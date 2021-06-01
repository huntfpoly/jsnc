# JavaScript Visualized: Event Loop

JavaScript là **single-threaded**: chỉ có thể chạy một tác vụ tại một thời điểm. Thông thường đó không phải là vấn đề lớn, nhưng bây giờ hãy tưởng tượng ta đang chạy một tác vụ mất 30 giây .. Mááá... Trong tác vụ đó, ta đang nhẫn nại đợi chờ hẳn 30 giây trước khi bất kỳ điều gì khác có thể xảy ra (JavaScript chạy trên chuỗi chính của trình duyệt theo mặc định, vì vậy toàn bộ giao diện người dùng bị đơ cmnl) 😬

May mắn vcl, trình duyệt giờ cung cấp cho ta một số tính năng mà bản thân bố JavaScript không cung cấp: **Web API**. Thằng này nó bao gồm **API DOM**, **setTimeout**, các yêu cầu **HTTP**, vân vân và mây mây... Điều này có thể giúp chúng tôi tạo ra một số hành vi **async** ( không đồng bộ), **non-blocking behavior** (hành vi không chặn - vl dịch), thôi dễ hiểu thì Blocking là code chạy tuần tự từ trên xuống, nếu đoạn nào chưa thực thi thì nó đứng cmn lại luôn, giống như gặp đèn đỏ có chốt ấy, nhưng anh JS lại như các tổ lái hồ Gươm, bố đ quan tâm và đưa ra cách non-blocking để nó thông mẹ chốt luôn, sau đó kết quả ra sao thì tính tiếp, tóm lại không để trình duyệt bị loading, ảnh hưởng trải nghiệm người dùng. 🚀

Khi ta gọi một hàm, nó sẽ được thêm vào một thứ gọi là **call stack** (dịch ra nó cứ bị sao ấy). **Call stack** là một phần của công cụ JS. Nó là stack, nó hoạt động theo cấu trúc **first in - last out** (nghĩa là vào trước , ra sau. nó giống như chui vào ống cống mà bịt mẹ 1 đầu vào ấy, ai chơi CF thì nó như cái cống ở Map Đấu Trường zombie :v ). Khi một hàm trả về một giá trị, nó sẽ được bật ra khỏi **stack**

![gid1.6.gif](https://firebasestorage.googleapis.com/v0/b/ecma-a7973.appspot.com/o/images%2Fgid1.6.gif?alt=media&token=ebaf8dee-6089-413f-a30a-5368da920bde)

Function`respond`trả về một hàm `setTimeout`. `SetTimeout` được cung cấp bởi `Web API`: nó cho phép duy trì các tác vụ mà không chặn luồng chính. Fucntion `callback` mà ta đã truyền cho hàm `setTimeout` là `arrow function` () => {return 'Hey'} được thêm vào Web API. Trong khi đó, hàm `setTimeout` và hàm response được bật ra khỏi stack, cả hai đều trả về giá trị!

![gif2.1.gif](https://firebasestorage.googleapis.com/v0/b/ecma-a7973.appspot.com/o/images%2Fgif2.1.gif?alt=media&token=ae82dccc-8537-431c-8c9c-c77ffd637162)

Trong Web API, khi function `setTimeout` chạy xong thì `callback` truyền vào không thực thi ngay mà nó chuyển vào `QUEUE`.

![gif3.1.gif](https://firebasestorage.googleapis.com/v0/b/ecma-a7973.appspot.com/o/images%2Fgif3.1.gif?alt=media&token=4c1c7890-f6ae-4725-b971-2bab5c92ac8a)

Đã đến lúc `event loop` thực hiện nhiệm vụ duy nhất của nó: **kết nối QUEUE với Call stack** Nếu callstack **trống**, vì vậy nếu tất cả các hàm được gọi trước đó đã trả về giá trị của chúng và đã được bật ra khỏi `callstack`, vị trí đầu tiên trong `QUEUE` sẽ được chuyển vào call stack.

![gif4.gif](https://firebasestorage.googleapis.com/v0/b/ecma-a7973.appspot.com/o/images%2Fgif4.gif?alt=media&token=e5474265-9d87-47a6-b262-c2d60f50c4aa)

function callback được thêm vào call stack, được thực thi và trả về giá trị, và bật ra khỏi stack

![gif5.gif](https://firebasestorage.googleapis.com/v0/b/ecma-a7973.appspot.com/o/images%2Fgif5.gif?alt=media&token=6c707659-0594-4966-af2b-8e78c06566b8)

Dựa vào kiến thức trên ta thử xét luồng chạy của đoạn code dưới đây xem như nào nhé;

```js
const foo = () => console.log("First");
const bar = () => setTimeout(() => console.log("Second"), 500);
const baz = () => console.log("Third");

bar();
foo();
baz();
```

nó sẽ xảy ra như sau:

![gif14.1.gif](https://firebasestorage.googleapis.com/v0/b/ecma-a7973.appspot.com/o/images%2Fgif14.1.gif?alt=media&token=935e90a7-5f57-441a-a82f-00fff1114d3c)

1. Đầu tiên ta gọi function `bar`.` bar` trả về `setTimeout` function.
2. Call back là `setTimeout` nên nó được thêm vào `Web API`, `setTimeout` function và `bar` được bật khỏi call stack.
3. timer trong `Web API` chạy , trong lúc chờ đợi thì function `foo` được gọi và logs ra `First`. `foo` trả về (undefined),function `baz` được gọi, và callback thêm vào `queue`.
4. `baz` logs ra `Third` . `event loop` thấy call stack trống sau khi baz được return, sau đó callback được đẩy vào call stack
5. callback logs ra `Second`.

Hy vọng rằng điều này làm cho bạn cảm thấy thoải mái hơn một chút với **Event loop**! Đừng lo lắng nếu nó vẫn có vẻ khó hiểu, điều quan trọng nhất là **hiểu được các lỗi / hành vi có thể đến từ đâu** để đưa lên **Google** và tìm những lỗi trong **Stack Overflow** trước 💪🏼 Khó hiểu mà không học thì nó còn khó hiểu hơn, hãy thực hành nhiều rồi các bug nó xảy ra, như thế sẽ hiểu rõ hơn về flow của nó, tóm lại trước mắt là phải bỏ thời gian ra học rồi quả chín rụng nhaaa...
