# Calendars and Timezones

## Calendar

Chrono is extended to support calendars and timezones. Only Gregorian calendar is supported.

```cpp
chrono::year_month_day date{2019y, September, 18d};
```
```cpp
auto date = 2019y/September/18d; // 18d/September/2019y || September/18d/2019y
```
```cpp
auto date = September/last/2019y;
```
```cpp
auto date = 31d/October/2019;
if (!date.ok()) {
  date = chrono::sys_days{date};
}
```
```cpp
auto date = Friday[2]/November/2019y;
```

## New duration type aliases

- `days`
- `weeks`
- `months`
- `years`

## New clocks

- `utc_clock` represents Coordinated Universal Time (UTC), measures time since
  00:00:00 UTC, Thursday, 1 January 1970, and includs leap seconds.

- `tai_clock` represents International Atomic Time (TAI), measures time since
  00:00:00, 1 January 1958, and was offseted 10 seconds ahead of UTC at that date, does not include leap seconds.

- `gps_clock` represents Global Positioning System (GPS) time, measures time since
  00:00:00, 6 January 1980 UTC, does not include leap seconds.

- `file_clock` - alias for the clock used for `filesystem::file_time_type::clock`, epoch is unspecified.

## New `system_clock`-related type aliases

```cpp
template<class Duration>
using sys_time = chrono::time_point<chrono::system_clock, Duration>;

using sys_seconds = sys_time<chrono::seconds>;
using sys_days = sys_time<chrono::days>;
```

```cpp
auto now = system_clock::now();
```

## Date + Time

```cpp
auto timestamp = sys_days{2019y/September/18d} + 9h + 35m + 10s; // 2019-09-18 09:35:10 UTC
```

## Timezone conversion

- Conver UTC to Denver time

```cpp
auto now = system_clock::now();
zoned_time current{current_zone(), now};
zoned_time denver{"America/Denver", now};
```

- Construct a localtime in Denver

```cpp
auto denver = chrono::zoned_time{"America/Denver", chrono::local_days{Wednesday[3]/September/2019} + 9h};
```

- Get current localtime

```cpp
auto local = chrono::zoned_time{chrono::current_zone(), chrono::system_clock::now()};
```

## Formating and parsing

C++20 `chrono` fully integrates into C++20 `format`

```cpp
auto tp = system_clock::now();
auto tz = locale_zone("Europe/Berlin");
cout << format("{:%F %T %Z}\n", zoned_time{tz, tp});
```
> **>_** 2019-11-14 12:13:14.123456 CET

```cpp
cout << format("{:%d.%m.%Y %T%z}\n", zoned_time{tz, tp});
```
> **>_** 14.11.2019 12:13:14.123556+0100

```cpp
cout << format(locale("de_DE"), "{:%d.%m.%Y %T%z}\n", zoned_time{tz, tp});
```
> **>_** 14.11.2019 12:13:14,123656+0100

```cpp
cout << format("{:%d.%m.%Y %T%z}\n", zoned_time{tz, floor<seconds>(tp)});
```
> **>_** 14.11.2019 12:13:14

If you can `format` it, you can `chrono::parse` it back in,
usually with the same formatting string

```cpp
system_clock::time_point tp;
cin >> parse("%d.%m.%Y %T%z", tp);
cout << tp << '\n';
```
> **>_**
```
input: 14.11.2019 12:13:14.123556+0100
output: 2019-11-14 11:13:14.123556
```

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>
