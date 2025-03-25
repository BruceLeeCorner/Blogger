# Utf8JsonWriter & Utf8JsonReader

Utf8JsonWriter和Utf8JsonReader用于生成和遍历Json字符串，步进单元如下枚举成员。

```c#
public enum JsonTokenType : byte
{
    // Summary:
    // There is no value (as distinct from System.Text.Json.JsonTokenType.Null). This
    // is the default token type if no data has been read by the System.Text.Json.Utf8JsonReader.
    None = 0,
    StartObject = 1,
    EndObject = 2,
    StartArray = 3,
    EndArray = 4,
    PropertyName = 5,
    Comment = 6,
    String = 7,
    Number = 8,
    True = 9,
    False = 10,
    Null = 11
}
```

相比于以string方式遍历或生成json字符串，JsonReader和JsonWriter更加简捷，不需要考虑各种转义，缩进等问题，下面的示例可以说明。

现生成Json片段`{"Name":"Jack","Age":17,"Skills":["C#","JavaScript"]}`

方式①  Utf8JsonWriter

```c#
var memoryStream = new MemoryStream();
Utf8JsonWriter writer = new Utf8JsonWriter(memoryStream);

writer.WriteStartObject();
writer.WritePropertyName("Name");
writer.WriteStringValue("Jack");
writer.WriteNumber("Age", 17);
writer.WritePropertyName("Skills");
writer.WriteStartArray();
writer.WriteStringValue("C#");
writer.WriteStringValue("JavaScript");
writer.WriteEndArray();
writer.WriteEndObject();
writer.Flush();
// Utf8JsonWriter的Write方法以UTF8编码将Json转换成字节并写入MemoryStream,所以解码时自然也要使用UTF8.
string json = Encoding.UTF8.GetString(memoryStream.ToArray()); 
```
方式② 手撸原生字符串

```c#
string json = "{" + "\"Name\":" + "\"Jack\"," + "\"Age\":" + "17," + "}" + ......
```

很显然，方式②很繁琐，需要自己处理字符转义，逗号等元素，特别容易出错。如果还要考虑构建带缩进的Json，方式②会更捉襟见肘了，而方式①，只需要搭配`JsonWriterOptions`设置项，就能作用到Writer实现对生成Json过程的细节控制，如缩进，即时校验。



`JsonReaderOptions`



# JsonConverter

## ColorJsonConverter

```c#
namespace System.Text.Json.Extensions.Converters
{

	public enum ColorStringFormatting
	{
		/// <summary>eg, #FF1234AB</summary>
		HexUppercaseAlpha,
		/// <summary>eg, #ff1234ab</summary>
		HexLowercaseAlpha,
		/// <summary>eg, #1234AB</summary>
		HexUppercase,
		/// <summary>eg, #1234ab</summary>
		HexLowercase,
		/// <summary>eg, 255,10,125,75</summary>
		RgbAlpha,
		/// <summary>eg, 10,125,75</summary>
		Rgb
	}

	public class ColorJsonConverter : JsonConverter<System.Windows.Media.Color>
	{
		public ColorJsonConverter()
		{
			ColorStringFormatting = ColorStringFormatting.HexUppercase;
		}

		public ColorStringFormatting ColorStringFormatting { get; set; }


		public override System.Windows.Media.Color Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
		{
			return (System.Windows.Media.Color)System.Windows.Media.ColorConverter
				.ConvertFromString(reader.GetString());
		}

		public override void Write(Utf8JsonWriter writer, System.Windows.Media.Color value, JsonSerializerOptions options)
		{
			if (ColorStringFormatting == ColorStringFormatting.HexUppercase)
			{
				writer.WriteRawValue($"\"#{value.R:X2}{value.G:X2}{value.B:X2}\"");
			}
			else if (ColorStringFormatting == ColorStringFormatting.HexUppercaseAlpha)
			{
				writer.WriteRawValue($"\"#{value.A:X2}{value.R:X2}{value.G:X2}{value.B:X2}\"");
			}

			else if (ColorStringFormatting == ColorStringFormatting.HexLowercase)
			{
				writer.WriteRawValue($"\"#{value.R:x2}{value.G:x2}{value.B:x2}\"");
			}
			else if (ColorStringFormatting == ColorStringFormatting.HexLowercaseAlpha)
			{
				writer.WriteRawValue($"\"#{value.A:x2}{value.R:x2}{value.G:x2}{value.B:x2}\"");
			}
			else if (ColorStringFormatting == ColorStringFormatting.Rgb)
			{
				writer.WriteRawValue($"\"{value.R},{value.G},{value.B}\"");
			}
			else if (ColorStringFormatting == ColorStringFormatting.RgbAlpha)
			{
				writer.WriteRawValue($"\"{value.A}{value.R}{value.G}{value.B}\"");
			}
			throw new ArgumentException(null, nameof(ColorStringFormatting));
		}
	}
}
```

## FontInfoJsonConverter

```c#
public class FontInfoJsonConverter : System.Text.Json.Serialization.JsonConverter<FontInfo>
{
    private Dictionary<string, System.Windows.FontStyle> _styles = new Dictionary<string, System.Windows.FontStyle>();

    public FontInfoJsonConverter()
    {
        _styles.Add(nameof(FontStyles.Normal), FontStyles.Normal);
        _styles.Add(nameof(FontStyles.Italic), FontStyles.Italic);
        _styles.Add(nameof(FontStyles.Oblique), FontStyles.Oblique);
    }

    public override FontInfo? Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
        {
            FontInfo? fontInfo = new FontInfo();
            while (reader.Read())
            {
                if (reader.TokenType == JsonTokenType.PropertyName)
                {
                    if (reader.GetString() == nameof(fontInfo.Size))
                    {
                        reader.Read();
                        fontInfo.Size = reader.GetInt32();
                    }
                    else if (reader.GetString() == nameof(fontInfo.Weight))
                    {
                        reader.Read();
                        fontInfo.Weight = FontWeight.FromOpenTypeWeight(reader.GetInt32());
                    }
                    else if (reader.GetString() == nameof(fontInfo.Stretch))
                    {
                        reader.Read();
                        fontInfo.Stretch = FontStretch.FromOpenTypeStretch(reader.GetInt32());
                    }
                    else if (reader.GetString() == nameof(fontInfo.Family))
                    {
                        reader.Read();
                        fontInfo.Family = new System.Windows.Media.FontFamily(reader.GetString());
                    }
                    else if (reader.GetString() == nameof(fontInfo.Style))
                    {
                        reader.Read();
                        fontInfo.Style = _styles[reader.GetString()!];
                    }
                    else if (reader.GetString() == nameof(fontInfo.BrushColor))
                    {
                        reader.Read();
                        fontInfo.BrushColor = new SolidColorBrush((System.Windows.Media.Color)System.Windows.Media.ColorConverter
                        .ConvertFromString(reader.GetString()));
                    }
                }
                else if (reader.TokenType == JsonTokenType.EndObject)
                {
                    break;
                }
            }
            return fontInfo;
        }

    public override void Write(Utf8JsonWriter writer, FontInfo value, JsonSerializerOptions options)
    {
        writer.WriteStartObject();
        writer.WritePropertyName(nameof(value.Size));
        writer.WriteNumberValue(value.Size);
        writer.WritePropertyName(nameof(value.Weight));
        writer.WriteNumberValue(value.Weight.ToOpenTypeWeight());
        writer.WritePropertyName(nameof(value.Stretch));
        writer.WriteNumberValue(value.Stretch.ToOpenTypeStretch());
        writer.WritePropertyName(nameof(value.Family));
        writer.WriteStringValue(value.Family.Source);
        writer.WritePropertyName(nameof(value.Style));
        writer.WriteStringValue(value.Style.ToString());
        writer.WritePropertyName(nameof(value.BrushColor));
        writer.WriteRawValue(System.Text.Json.JsonSerializer.Serialize(value.BrushColor.Color, options));
        writer.WriteEndObject();
    }
}
```









# Utf8JsonWriter

可以直接手写Json字符串，但是比较复杂，如 $"kjkjdosjojdsojodjsoj{Name}"，采用Utf8JsonWriter类，可以较为轻松的构建一个Json字符串。如JsonConverter的`void Write(Utf8JsonWriter writer, System.Windows.Media.Color value, JsonSerializerOptions options)`,可以写成`void Write(StringBuilder builder, System.Windows.Media.Color value, JsonSerializerOptions options)`,理论上这是可行的，但是builder太复杂了，对程序员不友好，所以采用了Utf8JsonWriter。

Utf8JsonWriter常用在JsonConverter中。



>Utf8JsonWriter的作用是逐步构造出一个合法的Json字符串，具有很高的性能。

> Indented 缩进
>
> SkipValidation 是否检查Json字符串合法
>
> WriteRawValue的SkipValidation一般设置成true比较好，也就是不检查。

```c#
var options = new JsonWriterOptions
{
    Indented = true,
    SkipValidation = false
};

using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream, options);

writer.WriteStartArray();

writer.WriteStartObject();
writer.WritePropertyName("Name");
writer.WriteStringValue("Jack");
writer.WritePropertyName("Age");
writer.WriteNumberValue(18);
writer.WritePropertyName("Gender");
writer.WriteBooleanValue(true);
writer.WritePropertyName("Mail");
writer.WriteNullValue();
writer.WriteEndObject();

writer.WriteStartObject();
writer.WritePropertyName("Name");
writer.WriteStringValue("Mary");
writer.WritePropertyName("Age");
writer.WriteNumberValue(17);
writer.WritePropertyName("Gender");
writer.WriteBooleanValue(false);
writer.WritePropertyName("Mail");
writer.WriteNullValue();
writer.WriteEndObject();

writer.WriteEndArray();

writer.Flush();
string json = Encoding.UTF8.GetString(stream.ToArray());
```

```json
[
  {
    "Name": "Jack",
    "Age": 18,
    "Gender": true,
    "Mail": null
  },
  {
    "Name": "Mary",
    "Age": 17,
    "Gender": false,
    "Mail": null
  }
]
```



---



```c#
var options = new JsonWriterOptions
{
    Indented = true,
    SkipValidation = false
};

using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream, options);

writer.WriteStartObject();
writer.WritePropertyName("Name");
writer.WriteStringValue("Jack");
writer.WritePropertyName("Skills");
writer.WriteStartArray();
writer.WriteStringValue("C#");
writer.WriteStringValue("Lua");
writer.WriteEndArray();
writer.WriteEndObject();

writer.Flush();
string json = Encoding.UTF8.GetString(stream.ToArray());
```

```json
{
  "Name": "Jack",
  "Skills": [
    "C#",
    "Lua"
  ]
}
```



---

> WriteBooleanValue,WriteStringValue,WriteNumberValue,WriteNullValue写入一个基元类型，WriteBoolean,WriteString,WriteNumber,WriteNull相当于WritePropertyName + Write...Value。WriteRawValue写入原生字符串，慎用。
>

Utf8JsonWriter每执行一次Write，其相应的Json字符串就会增长。使用WriteRawValue(string rawJson)时，要注意 ① 当前Utf8JsonWriter对应的Json字符串，保证rawJson续拼上去，是合法的Json格式；② 后续调用其他Write，续拼后同样是合法的Json格式。  如每次都会加个`,`,如果rawJson末尾写了`,`,则会导致最终的Json字符串格式错误。

如下实例，下面的Write序列执行完成，得到的并不是一个完成Json字符串。

```c#
writer.WriteStartObject();
writer.WriteNumber("temp",26);
writer.WritePropertyName("date");
writer.WriteEndObject();
```

```json
{
  "temp": 26,
  "date": 
}
```

可以用WriteRawValue补齐。

```c#
writer.WriteStartObject();
writer.WriteNumber("temp",26);
writer.WritePropertyName("date");
writer.WriteRawValue('"' + DateTimeOffset.UtcNow.ToString("yyyy-MM-dd HH:mm:ss") + '"');
writer.WriteEndObject();
```

```json
{
  "temp": 26,
  "date": "2024-07-29 14:50:54"
}
```