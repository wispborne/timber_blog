## How I Use Timber

#### Wait, what is Timber?
Timber is an open source logging library from Jake Wharton that provides more flexibility than Log. Instead of calling `Log.e`, call `Timber.e`. When you initialize Timber, you decide what to do with errors. If you simply want to wrap Log with no changes, you can initialize with the built-in `Timber.DebugTree`, which does nothing other than wrap Log. `Timber.v` calls `Log.v`. `Timber.d` calls `Log.d`. Etcetera. 
When you want to get fancy, and where Timber gets cool, is when you start making your own Trees (Tree? Timber? Logging? this is humor at it's finest).

#### Getting fancy
Do you leave your Log.v and Log.d calls in your app when you release it because you can't be bothered check for the build type whenever you write a log statement? I understand, but you're still a bad person. 

Enter Timber. Check this out:
```
if (isDebugMode()) { // or BuildConfig.DEBUG
    Timber.plant(new Timber.DebugTree());
} else {
    Timber.plant(new ReleaseLoggingTree());
    Crashlytics.start(context);
}
```
When you're developing, logs work as normal. Once is app is sent out into the wild, the `ReleaseLoggingTree` will be used instead.

`ReleaseLoggingTree` could look something like this, if you're using Crashlytics.
```public class CrashlyticsLoggingTree implements Timber.Tree {
    @Override
    public void v(String message, Object... args) {

    }

    @Override
    public void v(Throwable t, String message, Object... args) {

    }

    @Override
    public void d(String message, Object... args) {

    }

    @Override
    public void d(Throwable t, String message, Object... args) {

    }

    @Override
    public void i(String message, Object... args) {

    }

    @Override
    public void i(Throwable t, String message, Object... args) {

    }

    @Override
    public void w(String message, Object... args) {
        Log.w("WARN", message);
    }

    @Override
    public void w(Throwable t, String message, Object... args) {
        Log.w("WARN", message, t);
    }

    @Override
    public void e(String message, Object... args) {
        Crashlytics.log(message);
    }

    @Override
    public void e(Throwable t, String message, Object... args) {
        Crashlytics.logException(new Exception(message, t));

        try {
            Crashlytics.log(String.format(message, args));
        } catch (Exception e) {
            Log.e("", e.getMessage(), e);
        }
    }
}
```
