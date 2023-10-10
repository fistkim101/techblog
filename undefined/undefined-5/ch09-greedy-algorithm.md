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
package greedy.main4;

import java.util.*;

class Lecture implements Comparable<Lecture> {
    int money;
    int endTime;

    public Lecture(int money, int endTime) {
        this.money = money;
        this.endTime = endTime;
    }

    @Override
    public int compareTo(Lecture o) {
        return o.endTime - this.endTime;
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        List<Lecture> lectures = new ArrayList<>();
        int max = 0;
        for (int i = 0; i < n; i++) {
            Lecture lecture = new Lecture(scanner.nextInt(), scanner.nextInt());
            lectures.add(lecture);
            max = Math.max(max, lecture.endTime);
        }

        PriorityQueue<Integer> moneys = new PriorityQueue<>(Collections.reverseOrder());
        Collections.sort(lectures);
        int j = 0;
        int moneySum = 0;
        for (int i = max; i >= 1; i--) {
            for (; j < n; j++) {
                if (lectures.get(j).endTime < i) {
                    break;
                } else {
                    moneys.offer(lectures.get(j).money);
                }
            }
            if (!moneys.isEmpty()) {
                moneySum += moneys.poll();
            }
        }

        System.out.println(moneySum);
    }
}
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```java
package greedy.main6;

import java.util.Scanner;

public class Main {
    static int[] groups;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int memberCount = scanner.nextInt();
        int pairCount = scanner.nextInt();
        groups = new int[memberCount + 1];
        for (int i = 1; i <= memberCount; i++) {
            groups[i] = i;
        }
        for (int i = 0; i < pairCount; i++) {
            int firstMember = scanner.nextInt();
            int secondMember = scanner.nextInt();
            Union(firstMember, secondMember);
        }

        int target1 = scanner.nextInt();
        int target2 = scanner.nextInt();
        if (find(target1) == find(target2)) {
            System.out.println("YES");
        } else {
            System.out.println("NO");

        }
    }

    private static void Union(int firstMember, int secondMember) {
        int groupNumber1 = find(firstMember);
        int groupNumber2 = find(secondMember);
        if (groupNumber1 != groupNumber2) {
            groups[groupNumber1] = groupNumber2;
        }
    }

    private static int find(int member) {
        if (member == groups[member]) {
            return groups[member];
        } else {
            groups[member] = find(groups[member]);
            return groups[member];
        }
    }
}
```
