### SUGGESTION

**A DESCRIPTION OF THE PROBLEM :**
I think a lot of people write the following code to get tomorrow and yesterday.

```
LocalDate tomorrow = LocalDate.now().plusDays(1);
LocalDate yesterday = LocalDate.now().minusDays(1);
```

I think supporting tomorrow() and yesterday() functions will make it more convenient, meaningful, and more readable.

For example we can get 'tomorrow', 'yesterday' like following code.

```
LocalDate tomorrow = LocalDate.tomorrow(); // or LocalDate.tomorrow(zone); or LocalDate.tomorrow(clock);
LocalDate yesterday = LocalDate.yesterday(); // or LocalDate.yesterday(zone); or LocalDate.yesterday(clock);
```

Please review about this report.

<BR>

### DISCUSSION

```
This was not included in JSR-310 as it increases the potential for logical race conditions.

Consider a method:

boolean noTimeTravel() {
  return LocalDate.tomorrow.equals(LocalDate.today());
}

This method can sometimes return true!

The call to `tomorrow()` at 1ns before midnight will return tomorrows date (eg. 2022-06-02), but by the time `today()` is called it is now midnight and the date has changed thus `today()` returns 2022-06-02 as well. Since they are equal, true is returned.

In cases like this, developers are supposed to call `now()` only once per method (ideally once per logical unit of behaviour, such as a web request). By having a single method on each class that queries "now", it makes it easier to rationalise about the behaviour.

Thus, this suggestion should be rejected.
```


[JDK-8285775 : How about support LocalDate.tomorrow() and LocalDate.yesterday()](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=8285775)