# 해시

## 폰켓몬

```java
package programers.hash.main1;

import java.util.HashSet;
import java.util.Set;

class Solution {
    public int solution(int[] nums) {
        int selectCount = (nums.length / 2);
        Set<Integer> monsters = new HashSet<>();
        for (int monster : nums) {
            monsters.add(monster);
        }

        if (selectCount <= monsters.size()) {
            return selectCount;
        } else {
            return monsters.size();
        }
    }
}

class Main {
    public static void main(String[] args) {
        Solution solution = new Solution();
//        int[] nums = new int[]{3, 1, 2, 3}; // 2
//        int[] nums = new int[]{3,3,3,2,2,4}; // 3
        int[] nums = new int[]{3, 3, 3, 2, 2, 2}; // 2
        System.out.println(solution.solution(nums));
    }
}
```

## 완주하지 못한 선수

```java
package programers.hash.main2;

import java.util.HashMap;
import java.util.Map;

class Solution {
    public String solution(String[] participant, String[] completion) {
        Map<String, Integer> all = new HashMap<>();
        for (String player : participant) {
            all.put(player, all.getOrDefault(player, 0) + 1);
        }

        for (String player : completion) {
            if (all.containsKey(player)) {
                all.put(player, all.get(player) - 1);

                if (all.get(player) == 0) {
                    all.remove(player);
                }
            }
        }

        for (String player : all.keySet()) {
            return player;
        }

        return null;
    }
}

class Main {
    public static void main(String[] args) {
        Solution solution = new Solution();

        // "leo"
//        String[] participant = new String[]{"leo", "kiki", "eden"};
//        String[] completion = new String[]{"eden", "kiki"};

        // "vinko"
//        String[] participant = new String[]{"marina", "josipa", "nikola", "vinko", "filipa"};
//        String[] completion = new String[]{"josipa", "filipa", "marina", "nikola"};
//
//        // "mislav"
        String[] participant = new String[]{"mislav", "stanko", "mislav", "ana"};
        String[] completion = new String[]{"stanko", "ana", "mislav"};

        System.out.println(solution.solution(participant, completion));
    }
}
```

## 전화번호 목록

```java
package programers.hash.main3;

import java.util.Arrays;

class Solution {
    public boolean solution(String[] phone_book) {
        Arrays.sort(phone_book);
        for (int i = 0; i < phone_book.length - 1; i++) {
            String current = phone_book[i];
            String next = phone_book[i + 1];
            if (next.startsWith(current)) {
                return false;
            }
        }

        return true;
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
//        String[] phone_book = new String[]{"119", "97674223", "1195524421"}; // false
//        String[] phone_book = new String[]{"123","456","789"}; // true
        String[] phone_book = new String[]{"12", "123", "1235", "567", "88"}; // false
        System.out.println(T.solution(phone_book));
    }
}
```

## 의상

```java
class Solution {
    public int solution(String[][] clothes) {
        Map<String, Integer> cases = new HashMap<>();
        for (int i = 0; i < clothes.length; i++) {
            cases.put(clothes[i][1], cases.getOrDefault(clothes[i][1], 0) + 1);
        }

        int answer = 1;
        for (Integer value : cases.values()) {
            answer *= (value + 1);
        }

        return answer - 1;
    }
}
```



## 베스트 앨범

```java
package programers.hash.main5;

import java.util.*;
import java.util.stream.Collectors;

class Music implements Comparable<Music> {

    int index;
    int length;

    public Music(int index, int length) {
        this.index = index;
        this.length = length;
    }

    @Override
    public int compareTo(Music o) {
        if (o.length == this.length) {
            return this.index - o.index;
        }

        return o.length - this.length;
    }
}

class Solution {
    public int[] solution(String[] genres, int[] plays) {
        Map<String, Integer> genreMap = new HashMap<>();
        Map<String, PriorityQueue<Music>> musics = new HashMap<>();
        for (int i = 0; i < genres.length; i++) {
            String genre = genres[i];
            int length = plays[i];
            Music music = new Music(i, length);

            genreMap.put(genre, genreMap.getOrDefault(genre, 0) + length);
            if (!musics.containsKey(genre)) {
                musics.put(genre, new PriorityQueue<>());
            }
            musics.get(genre).offer(music);
        }

        List<String> sortedGenre = genreMap.entrySet()
                .stream()
                .sorted(Map.Entry.comparingByValue(Collections.reverseOrder()))
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
        List<Integer> answer = new ArrayList<>();
        for (String genre : sortedGenre) {
            int count = 0;
            while (!musics.isEmpty() && (!musics.get(genre).isEmpty()) && count < 2) {
                answer.add(musics.get(genre).poll().index);
                count++;
            }
        }

        return answer.stream().mapToInt(Integer::intValue).toArray();
    }
}

class Main {
    public static void main(String[] args) {
        Solution T = new Solution();
        int[] answer = T.solution(new String[]{"classic", "pop", "classic", "classic", "pop"}, new int[]{500, 600, 150, 800, 2500});
        for (int a : answer) {
            System.out.print(a + " ");
        }
    }
}
```

