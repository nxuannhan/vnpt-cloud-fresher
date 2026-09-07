# MARKDOWN

## Table of Contents

- [What is Markdown](#what-is-markdown)
- [Basic Syntax](#basic-syntax)
  1. [Heading](#21-heading)
  2. [](#22-paragraph)
  3. [](#23-line-breaks)
  4. [](#24-formatting)
  5. [](#25-blockquotes)
  6. [](#26-lists)
  7. [](#27-code)
  8. [](#28-horizontal-rules)
  9. [](#29-links)
  10. [](#210-images)
  11. [](#211-tables)
  12. [](#212-footnote)
  13. [](#213-heading-ids)
  14. [](#214-task-lists)
  15.

## 1. What is Markdown?

Markdown là ngôn ngữ đánh dấu nhẹ, sử dụng 

## 2. Basic Syntax

### 2.1. Heading

Thêm tiêu đề (heading) bằng cách thêm ký hiệu # trước một từ/cụm từ. Số lượng ký hiệu # tương đương với cấp (level) của tiêu đề.

# Heading level 1

## Heading level 2

### Heading level 3

### 2.2. Paragraph

Sử dụng một dòng trống để phân tách một hoặc nhiều đoạn văn bản.

I really like using Markdown. 

I'll use it to format all of my documents from now on.

### 2.3. Line Breaks

Ngắt dòng bằng hai dấu cách ở cuối dòng rồi Enter.

This is the first line.  
This is the second line.

### 2.4. Formatting

Để in nghiêng, thêm 1 ký hiệu * (hoặc _ ) vào trước và sau từ/cụm từ.

This is *italic text*. This is also _italic text_.

Thêm 2 ký hiệu * (hoặc _ ) vào trước và sau từ/cụm từ để bôi đậm.

This is **bold text**.
This is also __bold text__.

Để đồng thời bôi đậm và in nghiêng đoạn văn bản, sử dụng 3 ký hiệu * (hoặc _)

This is ***bold and italic text***.
This is also ___bold and italic text___.

Để gạch ngang nội dung, sử dụng hai ký tự ~ ở hai bên.

~~The world is flat.~~ We now know that the world is round.


### 2.5. Blockquotes

Để tạo một khối trích dẫn, thêm ký hiệu > trước đoạn văn bản.  
Blockquotes có thể chứa nhiều dòng. Sử dụng ký hiệu > với dòng trống giữa các đoạn văn bản để phân cách.  
Một blockquote có thể nằm lồng trong một blockquote khác. Sử dụng ký hiệu >> cho đoạn quote được lồng.

> This is a blockquote.
>
>> This is a nested blockquote.

### 2.6. Lists

Để tạo một danh sách có thứ tự (ordered list), thêm các mục trong danh sách với số thứ tự theo sau bởi dấu chấm. Danh sách nên bắt đầu với số 1.

1. First item
2. Second item
3. Third item
4. Fourth item

Để tạo một danh sách không theo thứ tự (unordered list), sử dụng ký hiệu gạch ngang (-), sao (*), hoặc cộng (+) trước các mục. Thụt lề vào trong để tạo danh sách lồng nhau.

- First item
- Second item
- Third item
    - Indented item
    - Indented item
- Fourth item

### 2.7. Code

Dùng một cặp ký hiệu bartick (`) để thể hiện inline code.

`npm install`

Dùng ba dấu bartick (```) ở đầu và cuối để biểu diễn khối code.

```javascript
function hello() {
    console.log("Hello World");
}
```

Phần `javascript` sau ký hiệu barticks (```) là languague identifier, hỗ trợ highlighting

Ngoài ra, có thể dùng ít nhất 4 spaces/1 tab để tạo code block.

    <html>
      <head>
        this is a code block
      </head>
    </html>

### 2.8. Horizontal Rules

Để tạo một đường kẻ ngang, sử dụng ba (hoặc nhiều hơn) dấu sao (***), gạch ngang (---) hoặc dấu gạch dưới (___) trên một dòng riêng.

***  

### 2.9. Links

Để tạo một liên kết, đặt văn bản hiển thị của liên kết trong dấu ngoặc vuông, đặt URL vào trong dấu ngoặc đơn.

[Google.com](https://www.google.com)

Có thể tùy chọn thêm tiêu đề (title) cho một liên kết. Tiêu đề xuất hiện dưới dạng tooltip khi di chuột lên liên kết. Để thêm tiêu đề, đặt tiêu đề trong dấu ngoặc kép ngay sau URL.

[Google.com](https://www.google.com "The best search engine")


Để biến một URL hoặc email thành link, đặt URL/email trong dấu ngoặc nhọn < >:

<https://www.google.com>

<example@email.com>

Một số trình xử lý Markdown (Markdown processor) hỗ trợ automatic linking / autolinking tự động chuyển các URL thành liên kết mà không cần đặt trong dấu ngoặc nhọn.

Nếu không muốn một URL tự động trở thành liên kết, định dạng URL dưới dạng code bằng dấu backtick (`).

`http://www.google.com`

**Reference-style link** là cách viết link thành 2 phần: Phần link nằm trong nội dung, phần chứa URL được định nghĩa ở nơi khác.  
Mục đích là làm Markdown dễ đọc hơn, đặc biệt khi có nhiều link hoặc URL dài.

[Google][1]

[1]: https://www.google.com 

### 2.10. Images

Để thêm hình ảnh, thêm dấu chấm than (!), theo sau là **alt text** đặt trong dấu ngoặc vuông, rồi đến **đường dẫn** (path hoặc URL) của hình ảnh trong dấu ngoặc đơn.

![Cats picture](https://cdn.britannica.com/34/235834-050-C5843610/two-different-breeds-of-cats-side-by-side-outdoors-in-the-garden.jpg)

### 2.11. Tables

Để tạo bảng, Markdown sử dụng:  
- pipe (|), dùng để ngăn cách các cột.  
- hyphens (---), dùng để tạo dòng phân cách giữa header và dữ liệu.

| Name | Description |
| --- | --- |
| Item | First line |
| Item | Second line |

Có thể căn nội dung trong cột bằng cách thêm dấu : vào dòng header separator

| Syntax | Description | Test Text |
| :--- | :----: | ---: |
| Left | Center | Right |

### 2.12. Footnote

Footnote cho phép thêm ghi chú hoặc tài liệu tham khảo. Khi tạo footnote, tại vị trí tham chiếu sẽ xuất hiện superscript. Người đọc có thể nhấp vào superscript để chuyển đến nội dung footnote ở phía dưới.

Here's a simple footnote,[^1] and here's a longer one.[^bignote]

[^1]: This is the first footnote.

[^bignote]: Here's one with multiple paragraphs and code.

    Indent paragraphs to include them in the footnote.

    `{ my code }`

    Add as many paragraphs as you like.

### 2.13. Heading IDs

Một số Markdown processor cho phép tự đặt ID cho heading. ID này được gắn vào HTML của heading và link trực tiếp đến một heading cụ thể.

[Heading IDs](#213-heading-ids)

### 2.14. Task Lists

Task list cho phép tạo danh sách các mục có checkbox.  
Một số Markdown processor hỗ trợ task list, checkbox sẽ được hiển thị bên cạnh nội dung.  
Tạo một task list bằng cách sử dụng dấu - và [ ] trước mỗi mục. Chọn một checkbox bằng cách thêm ký tự x vào giữa [ ].

- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media
