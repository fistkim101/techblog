# CH09 Greedy Algorithm

<figure><img src="../../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

```java
package greedy.main1;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

class Person implements Comparable<Person> {
    int height;
    int weight;

    public Person(int height, int weight) {
        this.height = height;
        this.weight = weight;
    }

    @Override
    public int compareTo(Person o) {
        return o.height - this.height;
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        final List<Person> persons = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            persons.add(new Person(scanner.nextInt(), scanner.nextInt()));
        }
        Collections.sort(persons);
        int max = Integer.MIN_VALUE;
        int count = 0;
        for (Person p : persons) {
            if (p.weight > max) {
                max = p.weight;
                count++;
            }
        }

        System.out.println(count);
    }
}
```

<figure><img src="../../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

```java
package greedy.main2;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

class Meeting implements Comparable<Meeting> {
    int start;
    int end;

    public Meeting(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    public int compareTo(Meeting o) {
        if (this.end != o.end) {
            return this.end - o.end;
        }

        return this.start - o.start;
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        final List<Meeting> meetings = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            meetings.add(new Meeting(scanner.nextInt(), scanner.nextInt()));
        }

        Collections.sort(meetings);
        int previousEnd = 0;
        int count = 0;
        for (Meeting m : meetings) {
            if (previousEnd <= m.start) {
                previousEnd = m.end;
                count++;
            }
        }
        System.out.println(count);
    }
}
```

<figure><img src="../../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

```java
package greedy.main3;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

class Time implements Comparable<Time> {
    int value;
    Character type;

    public Time(int value, Character type) {
        this.value = value;
        this.type = type;
    }

    @Override
    public int compareTo(Time o) {
        if (this.value == o.value) {
            return this.type - o.type;
        }
        return this.value - o.value;
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        List<Time> times = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            int start = scanner.nextInt();
            int end = scanner.nextInt();

            times.add(new Time(start, 's'));
            times.add(new Time(end, 'e'));
        }
        Collections.sort(times);
        int count = 0;
        int max = 0;
        for (Time t : times) {
            if (t.type == 's') {
                count++;
            } else {
                count--;
            }

            max = Math.max(count, max);
        }
        
        System.out.println(max);
    }
}
```

<figure><img src="../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

```java
```



























