<!-- This document is generated based on a corresponding .feature file, do not edit directly -->

# Plugin: Notifier (desktop notifications)

Desktop notifications can be enabled with the `:kaocha.plugin/notifier`
plugin. This will pop up a fail/pass notification bubble including a summary
of tests passed/errored/failed at the end of each test run. It's particularly
useful in combination with `--watch`, e.g. `bin/kaocha --plugin notifier
--watch`.

It does this by invoking a shell command that can be configured, so it can be
used to invoke an arbitrary command or script. By default, it will try to detect
which command to use, using either `notify-send` (Linux) or `terminal-notifier`
(Mac OS X), either of which may need to be installed first. If those commands
aren't available, it uses Java's included notifier. 

Several replacement patterns are available:

- `%{title}` : The notification title, either `⛔️ Failing` or `✅ Passing`
- `%{message}` : Test result summary, e.g., `5 tests, 12 assertions, 0 failures`
- `%{icon}` : Full local path to an icon to use (currently uses the Clojure icon)
- `%{failed?}` : `true` if any tests failed or errored, `false` otherwise
- `%{count}` : the number of tests
- `%{pass}` : the number of passing assertions
- `%{fail}` : the number of failing assertions
- `%{error}` : the number of errors
- `%{pending}` : the number of pending tests
- `%{urgency}` : `normal` if the tests pass, `critical` otherwise, meant for use with `notify-send`
- '%{timeout}` : configured timeout in milliseconds before the notification should disappear. By
    default, it passes -1, which corresponds to the default timeout when using
    `notify-send`.

Note that notifications don't time out on
[GNOME](https://gitlab.gnome.org/GNOME/gnome-shell/-/issues/112) and Ubuntu when
using [Notify
OSD](https://bugs.launchpad.net/ubuntu/+source/notify-osd/+bug/390508).

If no command is configured, and neither notification command is found, then
the plugin will print a warning. You can explicitly inhibit its behaviour
with `--no-notifications`.

You can combine the feature with the profiles feature:


``` clojure
{:plugins #profile {:default [:kaocha.plugin/notifier]
:ci []}}
```

This will silence the "Notification not shown because system does not support
it." warning.

## Enabling Desktop Notifications

- <em>Given </em> a file named "tests.edn" with:

``` clojure
#kaocha/v1
{:plugins [:kaocha.plugin/notifier]

 ;; Configuring a command is optional. Since CI does not have desktop
 ;; notifications we pipe to a file instead.
 :kaocha.plugin.notifier/command
 "sh -c 'echo \"%{title}\n%{message}\n%{failed?}\n%{count}\n%{urgency}\" > /tmp/kaocha.txt'"

 ;; Fallbacks:

 ;; :kaocha.plugin.notifier/command
 ;; "notify-send -a Kaocha %{title} %{message} -i %{icon} -u %{urgency}"

 ;; :kaocha.plugin.notifier/command
 ;; "terminal-notifier -message %{message} -title %{title} -appIcon %{icon}"
 }
```


- <em>And </em> a file named "test/sample_test.clj" with:

``` clojure
(ns sample-test
  (:require [clojure.test :refer :all]))

(deftest simple-fail-test
  (is (= :same :not-same)))
```


- <em>When </em> I run `bin/kaocha`

- <em>And </em> I run `cat /tmp/kaocha.txt`

- <em>Then </em> the output should contain:

``` nil
⛔️ Failing
1 tests, 1 failures.
true
1
critical
```



