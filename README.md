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

## 2. Nhiệm vụ cần làm

1. Tạo tay bài gồm năm lá. Constructor sắp xếp các lá theo hạng trước khi kiểm tra.
2. Dùng reflection trong mã test để gọi phương thức `private`, giữ nguyên mức truy cập trong `PokerHand`.
3. Kiểm tra tay bài hợp lệ, tay bài sai chất và tay bài sai hạng.
4. Đếm số lần gọi UUT cần cho statement coverage và branch coverage.

UUT trong bài này là `isStraightFlush()`. `Card`, `Rank` và `Suit` được lấy theo mã nguồn phiên bản 3 của repo. Phiên bản này dùng `rank()` và `suit()`, nên `PokerHand` gọi đúng hai accessor đó.

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

Điều kiện cần kiểm tra trong `isStraightFlush()` là:

```java
card.suit() != suit || card.rank().ordinal() - rank.ordinal() != 1
```

- Ở ca `True`, em dùng các lá 2, 3, 4, 5, 6 cùng chất tép. Chất không đổi và hạng tăng từng bậc, nên phương thức trả về `true`.
- Ở ca `WrongSuit`, em đổi lá 6 sang rô. Các hạng vẫn liên tiếp, nhưng chất khác nên phương thức trả về `false`.
- Ở ca `WrongRank`, em dùng 2, 3, 4, 5, 7 cùng chất tép. Hạng cuối bị đứt quãng nên phương thức trả về `false`.

### Số lần chạy tối thiểu

| Bài | Các ca cần chạy | Số lần gọi UUT | Mục tiêu |
|---|---|---:|---|
| 5.8 | `True` | 1 | Kiểm tra kết quả dương |
| 5.9 | `True` và một ca sai, ví dụ `WrongSuit` | 2 | Statement coverage đầy đủ |
| 5.10 | `True`, `WrongSuit`, `WrongRank` | 3 | Branch coverage đầy đủ cho điều kiện trong `isStraightFlush()` |

Trong `WrongSuit`, vế so sánh chất đúng nên phép `||` dừng tại đó. Trong `WrongRank`, vế chất sai và phép kiểm tra chuyển sang hạng. Ca `True` đi qua khi cả hai vế đều sai rồi đến `return true`. Em cần cả ba ca để kiểm tra các nhánh này ở bài 5.10.

Thuật toán nhận diện 9 dãy hạng liên tiếp cho mỗi chất. Có 4 chất nên cần 36 lần chạy để thử từng sảnh đồng chất. Các lần chạy đó chỉ kiểm tra tay bài cho kết quả `true`; chúng không bao phủ các tay bài cho kết quả `false`.

### Kết quả ghi nhận trong Eclipse

Các ảnh trong thư mục `img/` ghi lại ba lần chạy Coverage:

| Ảnh | Lần chạy | JUnit | Coverage `PokerHand.java` |
|---|---|---|---:|
| [`Screenshot 2026-10-25 231616.png`](img/Screenshot%202026-10-25%20231616.png) | Ca `True` | 3/3; 2 bỏ qua, 0 lỗi, 0 thất bại | 78/84 lệnh, 92,9% |
| [`Screenshot 2026-10-25 231917.png`](img/Screenshot%202026-10-25%20231917.png) | Ca sai chất | 3/3; 2 bỏ qua, 0 lỗi, 0 thất bại | 80/84 lệnh, 95,2% |
| [`Screenshot 2026-10-25 231946.png`](img/Screenshot%202026-10-25%20231946.png) | Chạy đủ ba test | 3/3; 0 bỏ qua, 0 lỗi, 0 thất bại | 80/84 lệnh, 95,2% |

Ở ảnh chạy cuối, toàn project đạt 94,1% instruction coverage. Với lớp `PokerHand.java`, Eclipse ghi 95,2% (80/84 lệnh); dòng `assert pCards.length == 5` trong constructor còn được đánh dấu một phần. Đây là số instruction coverage Eclipse hiển thị cho cả lớp; đề bài yêu cầu xét statement coverage ở bài 5.9 và branch coverage ở bài 5.10.

#### Ảnh 1. Chạy ca `True`

![JUnit và coverage cho ca True](img/Screenshot%202026-10-25%20231616.png)

#### Ảnh 2. Chạy ca sai chất

![JUnit và coverage cho ca sai chất](img/Screenshot%202026-10-25%20231917.png)

#### Ảnh 3. Chạy đủ ba test

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
