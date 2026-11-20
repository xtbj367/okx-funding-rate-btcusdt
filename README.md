# okx phí funding là gì: ai trả cho ai, bao lâu thu một lần, và lệnh 10 hợp đồng BTCUSDT bị trừ bao nhiêu? (bảng tính theo chu kỳ 1h/2h/4h/8h)

Phí funding trên OKX không phải khoản phí sàn thu của bạn. Tiền này đi thẳng từ ví của bên giữ vị thế này sang ví bên giữ vị thế đối diện, OKX chỉ làm trung gian chuyển và không giữ lại phần nào. Quy tắc đủ để nhớ trước khi đọc tiếp: funding rate dương thì bên long trả cho bên short, âm thì bên short trả cho bên long.

Một lệnh long 10 hợp đồng BTCUSDT ở mark price 60.000 USDT với funding rate 0,1% bị trừ 6 USDT mỗi kỳ. Bài này giải thích con số đó ra sao, kỳ thanh toán rơi vào giờ nào theo giờ Việt Nam, và vì sao hợp đồng chu kỳ 1 giờ không đắt gấp 8 lần hợp đồng 8 giờ.

> **Ba thứ cần xác nhận trước khi mở lệnh:** chu kỳ thanh toán của hợp đồng (mặc định 8 giờ, có hợp đồng 1, 2 hoặc 4 giờ), dấu của funding rate tại thời điểm đó, và mức cap/floor của chính hợp đồng ấy. Cả ba tra được trên trang Perpetual Contract Information hoặc qua API `GET /api/v5/public/funding-rate`. 👉 [Mở tài khoản OKX và kiểm tra funding rate của hợp đồng bạn định trade](https://okx.com/join/CASH20)

## Chu kỳ thanh toán: 8 giờ là mặc định, một số hợp đồng chạy 1, 2 hoặc 4 giờ

Mặc định OKX thu hoặc trả phí funding mỗi 8 giờ, rơi vào 00:00, 08:00 và 16:00 UTC. Đổi sang giờ Việt Nam là 07:00, 15:00 và 23:00.

| Chu kỳ | Mốc thanh toán (UTC) | Giờ Việt Nam (UTC+7) | Hệ số chia 8/N |  |
| --- | --- | --- | --- | --- |
| 8 giờ | 00:00, 08:00, 16:00 | 07:00, 15:00, 23:00 | ÷ 1 | [Xem](https://okx.com/join/CASH20) |
| 4 giờ | 00:00, 04:00, 08:00, 12:00, 16:00, 20:00 | 07:00, 11:00, 15:00, 19:00, 23:00, 03:00 | ÷ 2 | [Xem](https://okx.com/join/CASH20) |
| 2 giờ | mỗi 2 giờ, từ 00:00 đến 22:00 | mỗi 2 giờ, từ 07:00 đến 05:00 hôm sau | ÷ 4 | [Xem](https://okx.com/join/CASH20) |
| 1 giờ | mỗi giờ, từ 00:00 đến 23:00 | mỗi giờ | ÷ 8 | [Xem](https://okx.com/join/CASH20) |

Cột cuối ghi hệ số chia, không phải hệ số nhân. Đây là chỗ hay bị hiểu ngược nhất khi nhìn bảng: chu kỳ càng ngắn thì mỗi kỳ thu càng ít, không phải càng nhiều.

Công thức gốc trong tài liệu OKX cho ra một con số theo chuẩn 8 giờ, sau đó mới chia cho (8/N) để ra mức thu của từng kỳ. Với chu kỳ 1 giờ, funding rate hiển thị đã được chia 8 so với cùng mức premium index ở chu kỳ 8 giờ. Thời điểm thanh toán và mức cap/floor có thể được OKX điều chỉnh theo diễn biến thị trường, nên mốc cố định chỉ đúng cho tới lần điều chỉnh kế tiếp.

## Ai trả cho ai, và thời điểm nào thực sự bị trừ tiền

Funding rate dương: người giữ long trả cho người giữ short. Funding rate âm: người giữ short trả cho người giữ long. OKX không thu phí dịch vụ trong cơ chế này, toàn bộ tiền chuyển giữa hai bên.

Ba mốc thời gian dễ gây nhầm:

- Chỉ người còn giữ vị thế mở tại thời điểm đánh giá mới có nghĩa vụ trả hoặc quyền nhận.
- Đóng vị thế trước thời điểm đánh giá thì kỳ đó không bị trừ, cũng không được nhận.
- Quá trình đánh giá thực tế có thể kéo dài tới 1 phút. Mở lệnh lúc 00:00:20 UTC vẫn có thể bị tính phí nếu việc đánh giá chưa kết thúc. Việc tính toán diễn ra trong vài mili giây và không làm gián đoạn giao dịch.
- Nếu hợp đồng bị delist trước thời điểm đánh giá, kỳ funding đó bị hủy toàn bộ.

## Công thức funding rate: từng số hạng làm gì


Funding rate = clamp [ ( Average premium index
                        + clamp ( Interest rate − Average premium index, 0,05%, −0,05% ) )
                        / ( 8 / N ),
                        Funding rate cap, Funding rate floor ]


- **Interest rate = 0,01%**, cố định ở mọi chu kỳ thanh toán. Con số này không đổi dù hợp đồng chạy 1 giờ hay 8 giờ.
- **N** là chu kỳ thanh toán tính bằng giờ, nhận các giá trị 1, 2, 4, 8.
- **clamp(Interest rate − Average premium index, 0,05%, −0,05%)** giới hạn phần chênh giữa lãi suất và premium index trong khoảng ±0,05%. Chênh lệch lớn hơn bị cắt về đúng biên.
- **Chia (8/N)** là bước chuẩn hóa, để tổng chi phí một ngày xấp xỉ nhau giữa các chu kỳ.
- **clamp cuối cùng** chặn kết quả trong khoảng cap/floor của hợp đồng. Mức này khác nhau từng hợp đồng, tra trên trang funding của OKX.

Premium index đo độ lệch giữa giá thị trường và giá chỉ số:


Premium index = [ Max(0, Impact bid price − Index price)
                − Max(0, Index price − Impact ask price) ] / Index price


Impact bid/ask là giá khớp trung bình để khớp một khối lượng gọi là impact value, và impact value = 200 × đòn bẩy tối đa mà hợp đồng cho phép. Hợp đồng cho đòn bẩy càng cao thì khối lượng dùng để đo càng lớn, nên premium index của nó phản ứng mạnh hơn với cùng một độ sâu sổ lệnh.

Average premium index là trung bình động có trọng số của premium index trong kỳ trước, trọng số tăng dần theo thời gian: (1×T1 + 2×T2 + … + n×Tn) / (1+2+…+n). Chu kỳ 8 giờ lấy dữ liệu từng phút từ 00:00 đến 07:59, tức n = 480.

Funding rate dùng để tính phí là rate được tính ở phút cuối cùng trước khi đánh giá. Kỳ 16:00 dùng rate của 15:59, không dùng rate bạn thấy lúc 10 giờ sáng.

## Tự tính số tiền mình bị trừ

Công thức chung: **Phí funding = Giá trị vị thế × Funding rate**.

Giá trị vị thế tính khác nhau giữa hai loại hợp đồng:

- USDT-margined và USDC-margined: Giá trị vị thế = số hợp đồng × contract size × contract multiplier × mark price
- Coin-margined: Giá trị vị thế = số hợp đồng × contract size × contract multiplier / mark price

| Vị thế | Cách tính giá trị | Giá trị vị thế | Rate | Phí funding |
| --- | --- | --- | --- | --- |
| Long 10 hợp đồng BTCUSDT, mark 60.000 USDT, face 0,01 BTC/hợp đồng | 60.000 × 10 × 0,01 × 1 | 6.000 USDT | 0,1% | 6 USDT (trả) |
| Short 100 hợp đồng ETHUSD, mark 4.000 USD, face 10 USD/hợp đồng | 100 × 10 × 1 / 4.000 | 0,25 ETH | 0,1% | 0,00025 ETH (nhận) |

Hai ví dụ trên là ví dụ chính thức trong tài liệu OKX. Với funding rate dương, bên long ở dòng đầu mất 6 USDT, bên short ở dòng thứ hai nhận 0,00025 ETH. Cùng một mức rate 0,1% nhưng số tiền chênh nhau vì giá trị vị thế khác nhau, và đơn vị tiền cũng khác nhau.

Điểm cần lưu ý ở ví dụ thứ hai: hợp đồng coin-margined tính giá trị bằng cách chia cho mark price, nên khi giá ETH tăng, giá trị vị thế tính bằng ETH giảm xuống và phí funding cũng giảm theo.

👉 [Đăng ký OKX để thử tính phí funding trên hợp đồng bạn sắp mở](https://okx.com/join/CASH20)

## Ví dụ chu kỳ 1 giờ: vì sao 0,10% thành 0,00625%

Giả sử một hợp đồng có chu kỳ 1 giờ (N = 1) và average premium index của kỳ gần nhất là 0,10%.

1. Interest rate = 0,01%
2. 0,01% − 0,10% = −0,09%, bị clamp về −0,05%
3. Tử số: 0,10% + (−0,05%) = 0,05%
4. Funding rate = 0,05% / (8/1) = **0,00625% mỗi giờ** (giả định nằm trong cap/floor)

Cùng mức premium 0,10% đó ở chu kỳ 8 giờ: 0,05% / (8/8) = 0,05% mỗi 8 giờ.

So sánh chi phí một ngày: chu kỳ 1 giờ chạy 24 lần × 0,00625% = 0,15%. Chu kỳ 8 giờ chạy 3 lần × 0,05% = 0,15%. Bằng nhau. Con số 0,00625% trông rất nhỏ, nhưng nhân với 24 lần trong ngày thì ra đúng mức chi phí của hợp đồng 8 giờ.

Nếu kết quả vượt cap/floor của hợp đồng, nó bị chặn ở mức cap/floor chứ không giữ nguyên 0,00625%.

## Khi nào tần suất thanh toán tự nhảy lên 1 giờ

OKX tự điều chỉnh tần suất thanh toán của một hợp đồng theo diễn biến thị trường, và đưa về mặc định khi điều kiện bình thường trở lại. Cơ chế: mỗi lần funding rate chạm cap hoặc floor tại thời điểm thanh toán, tần suất tăng thêm một bậc.

- 8 giờ: 00:00, 08:00, 16:00 UTC
- 4 giờ: thêm các mốc 04:00, 12:00, 20:00
- 2 giờ: mỗi 2 giờ, từ 00:00 đến 22:00
- 1 giờ: mỗi giờ, từ 00:00 đến 23:00

Lần thanh toán đầu tiên sau khi đổi tần suất luôn rơi vào mốc kế tiếp trên bảng mới, không có kỳ thanh toán nào diễn ra ngoài các mốc đó.

Việc hạ tần suất chỉ được xét tại mốc thuộc lịch mặc định của hợp đồng. Nếu tại mốc đó, mọi kỳ thanh toán trong 12 giờ liền trước đều nằm trong khoảng ±0,20% (20 bps), OKX đưa tần suất về mặc định mà không thông báo thêm.

Ví dụ giả định trong tài liệu OKX: hợp đồng XYZUSDT có tần suất mặc định 4 giờ và cap/floor ±0,375%. Đây là tham số dùng cho ví dụ, không phải thông số của một hợp đồng thật. Kịch bản: 16:00 funding rate chạm 0,375% → nâng lên 2 giờ, kỳ tới 18:00. 18:00 chạm tiếp → nâng lên 1 giờ, kỳ tới 19:00. Đến 02:00 đã đủ điều kiện hạ, nhưng 02:00 không thuộc lịch 4 giờ nên OKX đợi tới 04:00 mới xét lại.

Hai ngoại lệ đáng biết: OKX có quyền tắt tính năng điều chỉnh tự động với từng hợp đồng cụ thể, và tính năng này chỉ áp dụng cho hợp đồng vĩnh viễn crypto (gồm X-Perps). Hợp đồng vĩnh viễn trên cổ phiếu, ETF hay hàng hóa luôn giữ tần suất mặc định, không bao giờ tự nhảy lên 1 giờ.

👉 [Xem tần suất thanh toán và cap/floor hiện tại của hợp đồng trên OKX](https://okx.com/join/CASH20)

## Rủi ro: phí funding có thể đẩy vị thế qua ngưỡng thanh lý

Phí bị trừ toàn bộ từ vị thế đang mở. Equity giảm sau khi trừ có thể kích hoạt giảm quy mô vị thế hoặc thanh lý.

Trang trợ giúp khu vực EEA của OKX mô tả rõ hơn bản toàn cầu: OKX thu đủ phần phí funding còn thiếu, kể cả khi việc thu này đẩy margin level xuống dưới 100%, rồi mới tiến hành thanh lý một phần hoặc toàn bộ nếu cần. Nói cách khác, phí funding được ưu tiên thu trước khi ngưỡng thanh lý được xử lý.

Cách trừ theo từng chế độ ký quỹ:

- **Ký quỹ riêng (isolated):** trừ từ margin của chính vị thế đó, không hủy lệnh chờ. Margin không đủ thì phần còn lại xử lý bằng thanh lý.
- **Ký quỹ chéo (single-currency, cross-currency, portfolio margin):** trừ từ equity của đồng tiền tương ứng trong tài khoản ký quỹ chéo, không hủy lệnh chờ.

Bên nhận được cộng toàn bộ vào margin của vị thế (isolated) hoặc vào equity đồng tiền tương ứng (cross).

Một tình huống thực tế: giữ lệnh long qua mốc 23:00 giờ Việt Nam với funding rate dương cao, tài khoản sát ngưỡng thanh lý. Khoản phí bị trừ có thể là giọt nước cuối cùng, và lệnh bị đóng ngay sau đó dù giá chưa đi ngược bao nhiêu.

## Trang OKX toàn cầu và trang EEA dùng hai cách viết công thức

Trang trợ giúp dành cho khu vực EEA của OKX vẫn trình bày công thức không có hệ số 8/N, với Interest rate = 0,03% / (24 / chu kỳ thanh toán). Với chu kỳ 8 giờ, lãi suất mỗi kỳ ra 0,01%, trùng kết quả với công thức toàn cầu. Khác biệt chỉ lộ ra ở chu kỳ 1, 2 và 4 giờ.

Bản toàn cầu, cập nhật ngày 28/10/2026, dùng Interest rate cố định 0,01% ở mọi chu kỳ và chia thêm (8/N). Nếu bạn đọc tài liệu ở trang EEA rồi áp sang hợp đồng chu kỳ ngắn, kết quả sẽ lệch. Lấy bản toàn cầu làm chuẩn.

Ngoài phí funding, giao dịch còn chịu phí giao dịch riêng theo bậc VIP, tính theo khối lượng giao dịch 30 ngày. Khoản đó nằm ở trang phí của OKX và không liên quan đến cơ chế funding ở đây, nhưng vẫn cần cộng vào chi phí thực khi tính lãi lỗ.

## Chọn theo kiểu giao dịch của bạn

Chỉ giao dịch các hợp đồng chạy chu kỳ 8 giờ mặc định: bạn chỉ cần canh ba mốc 07:00, 15:00 và 23:00 giờ Việt Nam. Muốn tránh kỳ tiếp theo thì đóng vị thế trước mốc đó, đừng mở lệnh mới trong khoảng một phút quanh mốc vì quá trình đánh giá có thể chưa kết thúc.

Giao dịch hợp đồng chu kỳ ngắn: funding rate hiển thị đã được chia 8 (với chu kỳ 1 giờ) hoặc chia 4 (với chu kỳ 2 giờ). Đừng nhìn con số nhỏ rồi giữ lệnh qua nhiều kỳ liên tiếp, hãy nhân với số kỳ trong ngày để ra chi phí thật. Và nhớ rằng tần suất có thể vừa được nâng lên 1 giờ sau một lần chạm cap mà bạn không nhận được thông báo nào.

Trước khi bấm mở lệnh, mở trang Perpetual Contract Information xem cap/floor và tần suất hiện tại của đúng hợp đồng đó. Cap/floor khác nhau từng hợp đồng, và funding rate thì đổi theo từng phút.

👉 [Bắt đầu trên OKX và tra funding rate trước khi vào lệnh](https://okx.com/join/CASH20)
