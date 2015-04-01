## Android Logging - Timber 101

#### Wait, what is Timber?
[Timber](https://github.com/JakeWharton/timber) is an open source logging library from [Jake Wharton](https://github.com/JakeWharton) at [Square](http://square.github.io/) (the company behind dagger, retrofit, picasso, and many more) that turns logging into a meaningful and useful debugging tool, even after app release. All calls to `Log.v/d/i/w/e(tag, message)` are replaced by `Timber.v/d/i/w/e(message)` or `Timber.v/d/i/w/e(exception, message)` is used. When you initialize Timber, you decide what to do with logs. If you simply want to wrap Log, you can use the built-in `Timber.DebugTree`, which calls Log and automagically uses the calling class name as the tag. `Timber.v` calls `Log.v`. `Timber.d` calls `Log.d`. Etcetera. 
When you want to get fancy, and where Timber gets cool, is when you start making your own Trees (Trees? Timber? Logging? this is humor at it's finest).

#### Getting fancy
Do you leave your Log.v and Log.d calls in your app when you release it because you can't be bothered check for the build type whenever you write a log statement? I understand, but you're still a bad person. 

Enter Timber. Check this out:
```java
if (isDebugMode()) { // BuildConfig.DEBUG
    Timber.plant(new Timber.DebugTree());
} else {
    Timber.plant(new ReleaseLoggingTree());
    Crashlytics.start(context);
}
```
When you're developing, logs work as normal. Once is app is sent out into the wild, the `ReleaseLoggingTree` will be used instead.

`ReleaseLoggingTree` could look something like this, if you're using Crashlytics.
```java
public class CrashlyticsLoggingTree implements Timber.Tree {
    @Override
    public void v(String message, Object... args) {    }

    @Override
    public void v(Throwable t, String message, Object... args) {    }

    @Override
    public void d(String message, Object... args) {    }

    @Override
    public void d(Throwable t, String message, Object... args) {    }

    @Override
    public void i(String message, Object... args) {    }

    @Override
    public void i(Throwable t, String message, Object... args) {    }

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
        Log.e("ERROR", args.length == 0 ? message : String.format(message, args));
        Crashlytics.log(message);
    }

    @Override
    public void e(Throwable t, String message, Object... args) {
        Log.e("ERROR", args.length == 0 ? message : String.format(message, args), t);
        Crashlytics.logException(new Exception(message, t));
    }
}
```

`Timber.v`, `Timber.d`, and `Timber.i` are all completely ignored. `Timber.w` logs only to logcat (opional), and `Timber.e` will actually send logged non-fatal errors to your Crashlytics dashboard in addition to logging to logcat.

#### Usage
Now that everything it in place, we have meaningful and useful logging levels to use throughout our app - `Timber.v` means verbose. `Timber.e` had better be important, because you'll be getting emails from Crashlytics about it.

```java
try {
    // Log in user
    Timber.v("User logged in, username=%s, url=%s, length=%d", username, url, response.length());
} catch (UnauthorizedException e) {
    throw new AuthenticationException("Authorization failure for " + username, e);
} catch (UnknownHostException e) {
    Timber.d(e, e.getMessage());
    throw e;
} catch (IOException e) {
    Timber.e(e, e.getMessage() + ", username=%s, url=%s", username, url);
    throw e;
} finally {
    // Close input stream
}
```

#### One more thing
Of course, Trees can be swapped at runtime. Enabling full logging is as simple as calling 
```java 
Timber.uprootAll(); // Removes all trees from Timber
Timber.plant(new ReleaseLoggingTree());
Timber.plant(new Timber.DebugTree());
```
Throw a checkbox onto the preferences screen to enable full logging, add a button to email logs to the dev, and you have yourself an easy way for the user to send the logcat for hard-to-reproduce bugs. 
