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

Trong ba bài này, em viết test cho lớp `PokerHand`, tập trung vào phương thức `isStraightFlush()`.

- **Bài 5.8:** Viết lớp test `TestPokerHand`, dùng reflection để gọi phương thức `private isStraightFlush()`, rồi kiểm tra một trường hợp có kết quả `true`. Đề cũng hỏi số lần chạy cần thiết để thử mọi sảnh đồng chất và liệu các lần chạy đó có kiểm thử hết mọi kiểu tay bài hay không.
- **Bài 5.9:** Mở rộng bộ test để đạt statement coverage đầy đủ với số lần gọi UUT ít nhất.
- **Bài 5.10:** Mở rộng bộ test để đạt branch coverage đầy đủ với số lần gọi UUT ít nhất.

Đề bài: [Chapter 5 Practice Exercises](https://github.com/prmr/DesignBook/blob/master/exercises/e-chapter5.md). Lời giải tham khảo cho ba bài: [Exercise 8–10](https://codesample.info/samples/designbook/solutions5/Exercise08to10.html).

## 2. Phân tích yêu cầu và cách làm

Trong các bài này, UUT (Unit Under Test) là phương thức `isStraightFlush()` của `PokerHand`. Phương thức được khai báo `private`, nên đề yêu cầu lớp test có helper để gọi nó bằng reflection. Em giữ nguyên mức truy cập của phương thức.

Constructor nhận năm lá và sắp xếp chúng theo hạng. `isStraightFlush()` lấy chất và hạng của lá đầu tiên, sau đó duyệt bốn lá còn lại. Mỗi lá phải cùng chất và có hạng cao hơn lá trước đúng một bậc. Gặp lá không thỏa điều kiện thì phương thức trả về `false`; nếu duyệt hết mà không gặp lỗi thì trả về `true`.

### Bài 5.8

Đề có hai yêu cầu. Trước tiên, viết `TestPokerHand` và kiểm tra một tay bài mà `isStraightFlush()` trả về `true`. Tiếp theo, tính số lần chạy cần thiết để thử mọi sảnh đồng chất, rồi phân biệt việc thử hết các trường hợp trả về `true` với việc kiểm thử hết mọi tay bài.

Helper `invokeIsStraightFlush` tìm phương thức bằng `getDeclaredMethod`, bật quyền truy cập bằng `setAccessible(true)`, rồi gọi phương thức trên đối tượng `PokerHand`. Test dùng các lá 2, 3, 4, 5, 6 cùng chất tép. Đây là sảnh đồng chất nên kết quả mong đợi là `true`.

Theo cách đếm của lời giải tham khảo, có 9 dãy hạng liên tiếp cho mỗi chất. Với 4 chất, cần 36 lần chạy để kiểm tra từng sảnh đồng chất. Những lần chạy này chỉ kiểm tra các tay bài trả về `true`; chúng không thể bao phủ tất cả tay bài trả về `false`.

### Bài 5.9

Đề yêu cầu mở rộng bộ test để đạt statement coverage đầy đủ với ít lần gọi UUT nhất. Một test `True` chạy qua vòng lặp và câu lệnh `return true`, nhưng chưa chạy câu lệnh `return false`. Vì vậy cần thêm một tay bài sai.

Trong project này, `WrongSuit` đổi lá 6 từ tép sang rô. Phương thức phát hiện chất khác và đi tới `return false`. Hai lần gọi UUT, gồm `True` và `WrongSuit`, là đủ cho statement coverage. Lời giải tham khảo cũng cho biết có thể dùng `WrongRank` thay cho `WrongSuit` ở bước này.

### Bài 5.10

Đề yêu cầu mở rộng bộ test để đạt branch coverage đầy đủ với ít lần gọi UUT nhất. Điều kiện trong phương thức dùng phép `||`:

```java
card.suit() != suit || card.rank().ordinal() - rank.ordinal() != 1
```

Để kiểm tra các nhánh của hai vế, cần ba tình huống. `True` làm cả hai vế sai. `WrongSuit` làm vế thứ nhất đúng, nên Java dừng đánh giá vế thứ hai. `WrongRank` làm vế thứ nhất sai rồi vế thứ hai đúng. Do hai trường hợp sai đi qua các nhánh khác nhau của phép `||`, bài 5.10 cần đủ ba lần gọi UUT.

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
| 5.8 | `True` | 1 | Kiểm tra kết quả dương |
| 5.9 | `True` và một ca sai, ví dụ `WrongSuit` | 2 | Statement coverage đầy đủ |
| 5.10 | `True`, `WrongSuit`, `WrongRank` | 3 | Branch coverage đầy đủ cho điều kiện trong `isStraightFlush()` |

Ba test đều chạy qua trong ảnh 3. Ca `True` trả về `true`; hai ca sai trả về `false`, như mong đợi.

### Kết quả ghi nhận trong Eclipse

#### Ảnh 1. Chạy ca `True`

JUnit báo 3/3, bỏ qua hai test; không có lỗi hay thất bại. Coverage của `PokerHand.java` là 78/84 lệnh, tương đương 92,9%.

![JUnit và coverage cho ca True](img/Screenshot%202026-10-25%20231616.png)

#### Ảnh 2. Chạy ca sai chất

JUnit báo 3/3, bỏ qua hai test; không có lỗi hay thất bại. Coverage của `PokerHand.java` là 80/84 lệnh, tương đương 95,2%.

![JUnit và coverage cho ca sai chất](img/Screenshot%202026-10-25%20231917.png)

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
