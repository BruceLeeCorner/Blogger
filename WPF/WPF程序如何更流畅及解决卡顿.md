1. ICommand的CanExecute是在UI线程上执行的，不能存在耗时操作。
2. ICommand的Execute是在UI线程上执行的，耗时任务必须放到其他非UI线程，使用await等待。
3. 单UI线程不够用时，可以多开几个UI线程。