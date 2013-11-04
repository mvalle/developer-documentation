<small>Piwik\ScheduledTime</small>

Weekly
======

Weekly class is used to schedule tasks every week.


Methods
-------

The class defines the following methods:

- [`getRescheduledTime()`](#getRescheduledTime)
- [`setDay()`](#setDay)
- [`getDayIntFromString()`](#getDayIntFromString)

### `getRescheduledTime()` <a name="getRescheduledTime"></a>

#### See Also

- `ScheduledTime::getRescheduledTime`

#### Signature

- It is a **public** method.
- It returns a(n) `int` value.

### `setDay()` <a name="setDay"></a>

#### Signature

- It is a **public** method.
- It accepts the following parameter(s):
    - `$day`
- It does not return anything.
- It throws one of the following exceptions:
    - [`Exception`](http://php.net/class.Exception) &mdash; if parameter _day is invalid

### `getDayIntFromString()` <a name="getDayIntFromString"></a>

#### Signature

- It is a **public static** method.
- It accepts the following parameter(s):
    - `$dayString`
- It does not return anything.
