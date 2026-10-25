# Post-Class 06 – Exercise 8, 9, 10: Kiểm thử PokerHand và độ phủ

| Thông tin | Nội dung |
|---|---|
| Họ và tên | Huỳnh Ngọc Khang |
| MSSV | 4051050116 |
| Lớp | CNTT46D |
| Bài | Post-Class 06 – Exercise 8, 9, 10 (Chapter 5, *Introduction to Software Design with Java*) |
| Project | `Postclass06` |
| Package | `e3.chapter5` |

## 1. Đề bài

`PokerHand` là lớp mẫu một phần của bài 3.12. Các yêu cầu được giao:

- **Bài 5.8:** Viết lớp test `TestPokerHand` với helper để kiểm tra phương thức private `isStraightFlush()`, rồi viết một test case cho kết quả `true`. Tính số lần thực thi UUT cần để kiểm tra mọi sảnh đồng chất có thể có và xác định các lần chạy đó có tạo thành exhaustive test coverage hay không.
- **Bài 5.9:** Mở rộng bộ test của `PokerHand` để đạt complete statement coverage với số lần thực thi UUT ít nhất.
- **Bài 5.10:** Mở rộng bộ test của `PokerHand` để đạt complete branch coverage với số lần thực thi UUT ít nhất.

Đề bài: [Chapter 5 Practice Exercises](https://github.com/prmr/DesignBook/blob/master/exercises/e-chapter5.md). Lời giải tham khảo cho ba bài: [Exercise 8–10](https://codesample.info/samples/designbook/solutions5/Exercise08to10.html).

## 2. Phân tích yêu cầu và cách làm

Trong các bài này, UUT (Unit Under Test) là phương thức `isStraightFlush()` của `PokerHand`. Phương thức được khai báo `private`, nên đề yêu cầu lớp test có helper để gọi nó bằng reflection. Em giữ nguyên mức truy cập của phương thức.

Project dùng JUnit Jupiter 5.10.2. Em chạy test trong Eclipse bằng **Run As → JUnit Test**. Để xem số liệu coverage, em chạy **Coverage As → JUnit Test** và đọc kết quả trong cửa sổ **Coverage**.

Constructor nhận năm lá và sắp xếp chúng theo hạng. `isStraightFlush()` lấy chất và hạng của lá đầu tiên, sau đó duyệt bốn lá còn lại. Mỗi lá phải cùng chất và có hạng cao hơn lá trước đúng một bậc. Gặp lá không thỏa điều kiện thì phương thức trả về `false`; nếu duyệt hết mà không gặp lỗi thì trả về `true`.

### Bài 5.8

Đề có hai yêu cầu. Trước tiên, viết `TestPokerHand` và kiểm tra một tay bài mà `isStraightFlush()` trả về `true`. Tiếp theo, tính số lần chạy cần thiết để thử mọi sảnh đồng chất, rồi phân biệt việc thử hết các trường hợp trả về `true` với việc kiểm thử hết mọi tay bài.

Test dùng là `testIsStraightFlush_True()`. Helper `invokeIsStraightFlush` tìm phương thức bằng `getDeclaredMethod`, bật quyền truy cập bằng `setAccessible(true)`, rồi gọi phương thức trên đối tượng `PokerHand`. Test tạo các lá `Card.get(Rank.TWO, Suit.CLUBS)` đến `Card.get(Rank.SIX, Suit.CLUBS)`. Kết quả mong đợi là `true`.

Theo cách đếm của lời giải tham khảo, có 9 dãy hạng liên tiếp cho mỗi chất. Với 4 chất, cần 36 lần chạy để kiểm tra từng sảnh đồng chất. Những lần chạy này chỉ kiểm tra các tay bài trả về `true`; chúng không thể bao phủ tất cả tay bài trả về `false`.

### Bài 5.9

Đề yêu cầu mở rộng bộ test để đạt statement coverage đầy đủ với ít lần gọi UUT nhất. Test `testIsStraightFlush_True()` chạy qua vòng lặp và câu lệnh `return true`, nhưng chưa chạy câu lệnh `return false`. Vì vậy cần thêm một test có kết quả `false`.

Trong project này, test `testIsStraightFlush_WrongSuit()` dùng cùng dãy hạng từ `Rank.TWO` đến `Rank.SIX`, nhưng lá cuối là `Card.get(Rank.SIX, Suit.DIAMONDS)`. Phương thức phát hiện giá trị `Suit` khác và đi tới `return false`. Hai lần gọi UUT, gồm `testIsStraightFlush_True()` và `testIsStraightFlush_WrongSuit()`, là đủ cho statement coverage. Lời giải tham khảo cũng cho biết có thể dùng `testIsStraightFlush_WrongRank()` thay cho test sai chất ở bước này.

### Bài 5.10

Đề yêu cầu mở rộng bộ test để đạt branch coverage đầy đủ với ít lần gọi UUT nhất. Điều kiện trong phương thức dùng phép `||`:

```java
card.suit() != suit || card.rank().ordinal() - rank.ordinal() != 1
```

Test `testIsStraightFlush_True()` làm cả hai vế sai. Test `testIsStraightFlush_WrongSuit()` làm vế thứ nhất đúng, nên Java dừng đánh giá vế thứ hai. Test `testIsStraightFlush_WrongRank()` dùng các lá `Rank.TWO`, `Rank.THREE`, `Rank.FOUR`, `Rank.FIVE`, `Rank.SEVEN`, tất cả có `Suit.CLUBS`; vế thứ nhất sai rồi vế thứ hai đúng. Ba lần gọi UUT bao phủ ba đường đánh giá của điều kiện `||`.

`Card`, `Rank` và `Suit` trong project theo phiên bản 3 của repo. `Card` phiên bản này có accessor `rank()` và `suit()`, nên các lời gọi trong `PokerHand` dùng đúng tên đó.

## 3. Code test

Test đầy đủ nằm tại `src/test/java/e3/chapter5/TestPokerHand.java`. Helper reflection và ba ca kiểm thử chính:

```java
private static boolean invokeIsStraightFlush(PokerHand pHand) {
    try {
        Method method = PokerHand.class.getDeclaredMethod("isStraightFlush");
        method.setAccessible(true);
        return (boolean) method.invoke(pHand);
    }
    catch (ReflectiveOperationException e) {
        e.printStackTrace();
        fail();
        return false;
    }
}

@Test
void testIsStraightFlush_True() {
    PokerHand hand = new PokerHand(Card.get(Rank.TWO, Suit.CLUBS),
            Card.get(Rank.THREE, Suit.CLUBS),
            Card.get(Rank.FOUR, Suit.CLUBS),
            Card.get(Rank.FIVE, Suit.CLUBS),
            Card.get(Rank.SIX, Suit.CLUBS));
    assertTrue(invokeIsStraightFlush(hand));
}

@Test
void testIsStraightFlush_WrongSuit() {
    PokerHand hand = new PokerHand(Card.get(Rank.TWO, Suit.CLUBS),
            Card.get(Rank.THREE, Suit.CLUBS),
            Card.get(Rank.FOUR, Suit.CLUBS),
            Card.get(Rank.FIVE, Suit.CLUBS),
            Card.get(Rank.SIX, Suit.DIAMONDS));
    assertFalse(invokeIsStraightFlush(hand));
}

@Test
void testIsStraightFlush_WrongRank() {
    PokerHand hand = new PokerHand(Card.get(Rank.TWO, Suit.CLUBS),
            Card.get(Rank.THREE, Suit.CLUBS),
            Card.get(Rank.FOUR, Suit.CLUBS),
            Card.get(Rank.FIVE, Suit.CLUBS),
            Card.get(Rank.SEVEN, Suit.CLUBS));
    assertFalse(invokeIsStraightFlush(hand));
}
```

## 4. Giải thích, kết quả và nhận xét

Kết quả kỳ vọng và số lần gọi UUT:

### Số lần chạy tối thiểu

| Bài | Các ca cần chạy | Số lần gọi UUT | Mục tiêu |
|---|---|---:|---|
| 5.8 | `testIsStraightFlush_True()` | 1 | Kiểm tra kết quả dương |
| 5.9 | `testIsStraightFlush_True()` và `testIsStraightFlush_WrongSuit()` | 2 | Statement coverage đầy đủ |
| 5.10 | `testIsStraightFlush_True()`, `testIsStraightFlush_WrongSuit()`, `testIsStraightFlush_WrongRank()` | 3 | Branch coverage đầy đủ cho điều kiện trong `isStraightFlush()` |

Ảnh 3 xác nhận các kết quả mong đợi: `testIsStraightFlush_True()` đạt `assertTrue`; `testIsStraightFlush_WrongSuit()` và `testIsStraightFlush_WrongRank()` đạt `assertFalse`.

### Kết quả ghi nhận trong Eclipse

#### Ảnh 1. Chạy `testIsStraightFlush_True()`

JUnit báo 3/3, bỏ qua hai test; không có lỗi hay thất bại. Coverage của `PokerHand.java` là 78/84 lệnh, tương đương 92,9%.

![JUnit và coverage cho testIsStraightFlush_True](img/Screenshot%202026-10-25%20231616.png)

#### Ảnh 2. Chạy `testIsStraightFlush_WrongSuit()`

JUnit báo 3/3, bỏ qua hai test; không có lỗi hay thất bại. Coverage của `PokerHand.java` là 80/84 lệnh, tương đương 95,2%.

![JUnit và coverage cho testIsStraightFlush_WrongSuit](img/Screenshot%202026-10-25%20231917.png)

#### Ảnh 3. Chạy đủ ba test

JUnit báo 3/3, không bỏ qua test nào; không có lỗi hay thất bại. Coverage của `PokerHand.java` là 80/84 lệnh, tương đương 95,2%; toàn project đạt 94,1%. Dòng `assert pCards.length == 5` trong constructor còn được đánh dấu một phần. Đây là số instruction coverage Eclipse hiển thị cho cả lớp; đề bài yêu cầu xét statement coverage ở bài 5.9 và branch coverage ở bài 5.10.

![JUnit và coverage sau khi chạy đủ ba test](img/Screenshot%202026-10-25%20231946.png)

## 5. Cấu trúc project và nguồn

```text
Postclass06/
├── img/
│   ├── Screenshot 2026-10-25 231616.png
│   ├── Screenshot 2026-10-25 231917.png
│   └── Screenshot 2026-10-25 231946.png
├── pom.xml
└── src/
    ├── main/java/e3/chapter5/
    │   ├── Card.java
    │   ├── PokerHand.java
    │   ├── Rank.java
    │   └── Suit.java
    └── test/java/e3/chapter5/
        └── TestPokerHand.java
```

Project dùng JUnit Jupiter 5.10.2 và mức tương thích Java 8.

Các lớp `Card`, `Rank` và `Suit` trong package `e3.chapter5` được lấy từ mã nguồn đi kèm sách *Introduction to Software Design with Java* của Martin P. Robillard, phiên bản 3. Có thể tìm chúng trong thư mục [`code-e3/e3/chapter5`](https://github.com/prmr/DesignBook/tree/master/code-e3/e3/chapter5) của [repository DesignBook](https://github.com/prmr/DesignBook).

Lớp `PokerHand` dựa trên phần mã mẫu trong [đề bài chương 5](https://github.com/prmr/DesignBook/blob/master/exercises/e-chapter5.md). Em dùng tên accessor `rank()` và `suit()` theo `Card` phiên bản 3. Các trường hợp kiểm thử được đối chiếu với [lời giải tham khảo bài 5.8–5.10](https://codesample.info/samples/designbook/solutions5/Exercise08to10.html).
