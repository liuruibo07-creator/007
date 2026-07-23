# JavaSE进阶练习答案

## 一、单选题

### 1. 答案：B

Java 多态的条件包括：有继承或实现关系、子类重写父类方法、父类引用指向子类对象。方法重载不属于运行时多态的必要条件。

### 2. 答案：A

`try` 后通常使用 `catch` 处理异常，也可以只配合 `finally` 使用。`finally` 不是必须的；释放资源通常放在 `finally` 中或使用 try-with-resources。

### 3. 答案：B

`new FileOutputStream("aa.txt")` 默认覆盖原文件，执行后文件内容变为“我爱学习”。

### 4. 答案：D

非静态同步方法的锁对象是当前对象 `this`；静态同步方法的锁对象是当前类的字节码对象，如 `Student.class`。

### 5. 答案：C

抽象类有构造方法，接口没有构造方法。抽象类可以包含普通成员变量和普通方法；类可以实现多个接口。

### 6. 答案：C

`new FileWriter("a.txt", true)` 中的第二个参数为 `true`，表示以追加方式写入。

### 7. 答案：D

代码调用的是 `t.run()`，但 `MyThread` 没有重写 `run()`。它重写的 `start()` 不会被调用，因此程序编译通过且没有输出。

### 8. 答案：B

`Reader` 没有 `read(int n)` 这个读取单个字符的重载。读取单个字符应调用无参数的 `read()`。

### 9. 答案：C

访问范围从大到小为：`public > protected > 默认（包访问权限） > private`。

### 10. 答案：A

Map 的键不能重复，值可以重复。放入相同键时，新值会替换旧值。

### 11. 答案：A

网络编程三要素是 IP 地址、端口和协议，域名不属于三要素。

### 12. 答案：C

`keepAliveTime` 的时间单位由 `TimeUnit` 参数指定，不一定是秒。

### 13. 答案：C

`Collections.sort(list, (x, y) -> y - x)` 按降序排列。实际开发中，为避免整数减法溢出，更推荐：

```java
list.sort(Comparator.reverseOrder());
```

### 14. 答案：C

`contains("java")` 区分大小写，匹配到 3 个元素：`java入门到精通.pdf`、`aaa.java`、`java疯狂笔记.txt`。跳过 1 个后剩 2 个。

### 15. 答案：D

JDK 8 中 HashMap 的底层结构是数组 + 链表 / 红黑树。链表达到树化条件时会转为红黑树。

## 二、多选题

### 16. 答案：B、C

- A 正确：`read(char[] cbuf)` 返回实际读取的字符数，到达文件末尾时返回 `-1`。
- B 错误：`FileWriter` 支持 `FileWriter(String fileName, boolean append)`，可直接追加写入。
- C 错误：`FileReader` 本身没有读取一整行的方法，通常用 `BufferedReader.readLine()`。
- D 正确：`FileReader` 没有类似 `FileInputStream.available()` 的方法来获取待读取字节数，而且字符数也不一定等于字节数。

### 17. 答案：A、B、C、D

四个方法的描述均正确。

### 18. 答案：A、C

常见的多线程实现方式包括继承 `Thread` 类、实现 `Runnable` 接口、实现 `Callable` 接口并配合 `FutureTask` 使用。原题 E 写的是不存在的 `Call` 接口，因此按题面不能选择 E；如果题目原意是 `Callable`，则 E 也应选择。

### 19. 答案：B、D

B 通过 `keySet()` 遍历键，再根据键取值；D 通过 `entrySet()` 同时取得键和值。两种写法都能正确遍历 Map。

### 20. 答案：A、C、D

- A 正确：可通过 `Class.forName("类的全限定名")` 获取 Class 对象。
- B 错误：获取私有方法应使用 `getDeclaredMethod()`，并通常需要调用 `setAccessible(true)`。
- C 正确：`getConstructor()` 可获取公共无参数构造方法。
- D 正确：可使用 `method.invoke(对象, 参数...)` 调用方法。

## 三、代码题

### 21. 统计单词出现次数并按键的自然顺序输出

使用 `TreeMap` 存储数据，可以自动按照单词的自然顺序排列键。

```java
import java.util.Map;
import java.util.TreeMap;

public class WordCountDemo {
    public static void main(String[] args) {
        String text = "wo ai heima wo ai java wo yao hao hao xue xi java";

        Map<String, Integer> wordCount = new TreeMap<>();
        String[] words = text.split("\\s+");

        for (String word : words) {
            wordCount.put(word, wordCount.getOrDefault(word, 0) + 1);
        }

        StringBuilder result = new StringBuilder();
        for (Map.Entry<String, Integer> entry : wordCount.entrySet()) {
            result.append(entry.getKey())
                  .append('(')
                  .append(entry.getValue())
                  .append(')');
        }

        System.out.println(result);
    }
}
```

运行结果：

```text
ai(2)hao(2)heima(1)java(2)wo(3)xi(1)xue(1)yao(1)
```

### 22. 录入学生信息，按总分降序写入文件并读取显示

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.Scanner;

public class StudentScoreDemo {
    private static final String FILE_NAME = "students.txt";

    public static void main(String[] args) throws IOException {
        List<Student> students = inputStudents();

        // 总分从高到低排序；总分相同时按姓名自然顺序排序。
        students.sort(
                Comparator.comparingInt(Student::getTotalScore)
                          .reversed()
                          .thenComparing(Student::getName)
        );

        writeStudents(students);
        readAndPrintStudents();
    }

    private static List<Student> inputStudents() {
        Scanner scanner = new Scanner(System.in);
        List<Student> students = new ArrayList<>();

        for (int i = 1; i <= 3; i++) {
            System.out.println("请输入第 " + i + " 个学生的信息：");
            System.out.print("姓名：");
            String name = scanner.nextLine().trim();

            int chinese = readScore(scanner, "语文成绩：");
            int math = readScore(scanner, "数学成绩：");
            int english = readScore(scanner, "英语成绩：");

            students.add(new Student(name, chinese, math, english));
        }
        return students;
    }

    private static int readScore(Scanner scanner, String prompt) {
        while (true) {
            System.out.print(prompt);
            String input = scanner.nextLine().trim();
            try {
                int score = Integer.parseInt(input);
                if (score >= 0 && score <= 100) {
                    return score;
                }
            } catch (NumberFormatException ignored) {
                // 输入不合法时继续提示。
            }
            System.out.println("成绩必须是 0 到 100 之间的整数，请重新输入。");
        }
    }

    private static void writeStudents(List<Student> students) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_NAME))) {
            for (Student student : students) {
                writer.write(student.toFileString());
                writer.newLine();
            }
        }
    }

    private static void readAndPrintStudents() throws IOException {
        System.out.println("\n文件中的学生信息如下：");
        try (BufferedReader reader = new BufferedReader(new FileReader(FILE_NAME))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }
    }

    private static class Student {
        private final String name;
        private final int chinese;
        private final int math;
        private final int english;

        Student(String name, int chinese, int math, int english) {
            this.name = name;
            this.chinese = chinese;
            this.math = math;
            this.english = english;
        }

        String getName() {
            return name;
        }

        int getTotalScore() {
            return chinese + math + english;
        }

        String toFileString() {
            return name + "," + chinese + "," + math + "," + english;
        }
    }
}
```

例如录入林青霞的三科成绩后，文件中对应一行的格式为：

```text
林青霞,98,99,100
```
