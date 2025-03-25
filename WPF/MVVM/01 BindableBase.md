## BindableBase

> Prism提供BindableBase作为ViewModel的基类。

个人认为Prism的BindableBase不如CommunityToolkit.Mvvm的ObservableObject功能丰富和强大。如：
- SetProperty只支持back-field memory Property，不支持non-back-field Computed Property。
- SetProperty不支持自定义判等器。
- 不支持Task属性完成时二次通知。

下面展示BindableBase的源码
```csharp
public abstract class BindableBase : INotifyPropertyChanged
{
        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual bool SetProperty<T>(ref T storage, T value, [CallerMemberName] string propertyName = null)
        {
            if (EqualityComparer<T>.Default.Equals(storage, value)) return false;

            storage = value;
            RaisePropertyChanged(propertyName);

            return true;
        }

        protected virtual bool SetProperty<T>(ref T storage, T value, Action onChanged, [CallerMemberName] string propertyName = null)
        {
            if (EqualityComparer<T>.Default.Equals(storage, value)) return false;

            storage = value;
            onChanged?.Invoke();
            RaisePropertyChanged(propertyName);

            return true;
        }

        protected void RaisePropertyChanged([CallerMemberName] string propertyName = null)
        {
            OnPropertyChanged(new PropertyChangedEventArgs(propertyName));
        }

        protected virtual void OnPropertyChanged(PropertyChangedEventArgs args)
        {
            PropertyChanged?.Invoke(this, args);
        }
}
```