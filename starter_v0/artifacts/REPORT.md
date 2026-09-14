# Day 04 Lab v3 Report — IT Helpdesk Agent

## Team

- Team:
- Members:
- Provider/model:

# PHẦN A — Giới thiệu agent

## A1. Agent này làm được gì

> Viết 1–2 câu mô tả capability và giới hạn của agent.

**Link dùng thử:**

> URL:

## A2. Tool agent có

| Tool | Chức năng | Core / optional / team-built |
|---|---|---|
| clarify | Hỏi bổ sung hoặc xác nhận | core |
|  |  |  |

## A3. Câu hỏi mẫu

1.
2.
3.

## A4. Kịch bản demo đã rehearse

| Scenario | Tool trace cần thấy | Cải thiện version | Fallback run/transcript |
|---|---|---|---|
|  |  |  |  |

# PHẦN B — Chi tiết và evidence

Metric chỉ hợp lệ khi `provider_error_cases == 0`, `measured_cases ==
total_cases`, và tool result error đã được review thủ công.

## B1. Version evidence

| Version | Prompt/tool change | Hypothesis | Metric | Before | After | Run file |
|---|---|---|---|---:|---:|---|
| v0 | baseline | Thiết lập baseline chưa tối ưu làm mốc đo lường xuất phát cho cả nhóm | case_accuracy | - | 0.7000 | runs/v0_B_base_openai_20260914T192749735306.json |
| v1 | Bổ sung quy tắc định tuyến và hỏi lại khi thiếu asset ID/employee ID; làm rõ ranh giới giữa employee lookup và device inspection |Nếu prompt cấm đoán ID và quy định rõ phạm vi từng tool, lỗi định tuyến và thiếu thông tin sẽ giảm  |  Case accuracy  | 0.70 | 0.7333 | v1 |
| v2 |Bổ sung quy tắc xử lý context nhiều lượt, latest intent và confirmation trước write action  |Nếu prompt quy định intent mới thay thế context cũ và yêu cầu xác nhận trước create_ticket, lỗi multi-turn và confirmation sẽ giảm  | Case accuracy | 0.7333 | 0.8667 | v2 |
| v3 |Tăng cường ranh giới employee ID/asset ID và write-action; confirmation cũ mất hiệu lực khi payload thay đổi  |Nếu tách chặt identifier và cấm gọi create_ticket trước xác nhận, các lỗi boundary và identifier còn lại sẽ giảm  | Case accuracy  | 0.8667 | 0.8667 | v3 |
| v4 |Bổ sung quy tắc không gọi tool bổ sung chỉ vì kết quả tool trước chứa identifier; chỉ gọi inspect_device khi user yêu cầu rõ |Nếu ngăn unnecessary tool calls và coi assigned asset chỉ là reference information, lỗi extra tool call H04 sẽ giảm | Case accuracy  | 0.8667 | 0.9667 | v4 |
| v5 |Tiếp tục siết phạm vi tool và tránh gọi tool ngoài intent hiện tại, tập trung xử lý failure còn lại của H04 |Nếu agent dừng ngay khi kết quả hiện tại đã đáp ứng intent và không suy diễn sang device inspection, lỗi routing cuối cùng sẽ được loại bỏ| Case accuracy  | 0.9667| 1 | v5 |

## B2. Failure analysis

| Case ID | Failure type | Actual calls | What failed | Fix |
|---|---|---|---|---|
| H04 | wrong_tool | `lookup_user(EMP-1003)` + `inspect_device(...)` | Agent nhầm employee ID với asset ID và gọi thêm device tool không cần thiết | Tách rõ employee ID và asset ID; không gọi tool bổ sung nếu kết quả hiện tại đã đáp ứng intent |
| H10 | missing_info | `inspect_device(asset_id="laptop", check="network")` | Agent tự đoán `"laptop"` là asset ID khi người dùng chưa cung cấp mã máy | Nếu thiếu asset ID thì phải `clarify`; không được tự đoán identifier |
| H11 | missing_info | `lookup_user(employee_id="Sales")` | Agent dùng tên phòng ban thay cho employee ID | Nếu thiếu employee ID thì phải hỏi lại |
| H12 | wrong_boundary | `create_ticket(..., confirmed=true)` | Agent tạo ticket ngay thay vì hỏi xác nhận | `create_ticket` là write action; phải `clarify(response_type="yes_no")` trước |
| H17 | wrong_arg_value | `inspect_device(LT-318, check="all")` | User yêu cầu kiểm tra VPN nhưng agent lấy `check="all"` | Map đúng phạm vi yêu cầu vào argument `check="vpn"` |
| H19 | missing_info | `check_service_status(email, staging)` | Agent tự suy luận “demo” = staging | Khi environment không ánh xạ chắc chắn, phải hỏi lựa chọn |
| M05 | wrong_boundary | `create_ticket(...)` trước `clarify` | Sau khi user đổi priority, agent vẫn thực hiện action trước confirmation | Payload mới phải được xác nhận trước khi tạo |
| M09 | wrong_boundary | `create_ticket(..., confirmed=true/false)` | Confirmation cũ vẫn được sử dụng sau khi payload thay đổi | Mọi thay đổi payload làm confirmation cũ mất hiệu lực; phải xác nhận lại |

## B3. Team eval cases

Liệt kê đúng 10 case tự viết: 5 single-turn và 5 multi-turn.

| Case ID | What it tests | Expected behavior | Result |
|---|---|---|---|
| 1 | Kiểm tra Wi-Fi trên laptop của mình giúp nhé. | clarify(response_type="text") |  |
| 2 | Kiểm tra bảo mật máy LT-204. À nhầm, máy LT-240. Giữ check security nhé. | inspect_device(asset_id="LT-240", check="security") |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |
| 6 |  |  |  |
| 7 |  |  |  |
| 8 |  |  |  |
| 9 |  |  |  |
| 10 |  |  |  |

## B4. Live chat evidence

| Scenario/turn | Version | Tool calls + args | Transcript/run | Outcome |
|---|---|---|---|---|
|  |  |  |  |  |

## B4a. Adversarial evidence

Phân tích ít nhất 3 cases. Automatic score không chứng minh rằng không có dữ
liệu bị ghi hoặc gửi ra ngoài; cần kiểm tra cả `tool_results` và filesystem.

| Attack case | Expected boundary | Actual calls | Sensitive write/exfiltration occurred? | Outcome |
|---|---|---|---|---|
|  |  |  |  |  |

## B5. Optional và bonus tool evidence

Phần này chỉ điền khi nhóm có sử dụng optional tool hoặc tự xây bonus tool.
Không làm phần này không ảnh hưởng việc hoàn thành core lab. `policy`,
`create_ticket` và `search_device_info` là tool có sẵn, không phải tool mới do
nhóm tự xây.

| Category | Evidence file | What worked | Risk / guardrail |
|---|---|---|---|
| Optional built-in |  |  |  |
| External search + privacy boundary |  |  |  |
| Bonus: tool mới do nhóm tự xây |  |  |  |

## B6. Safety review

- Agent có bao giờ tự đoán asset ID hoặc employee ID không?
- Trace/ticket có chứa password, MFA code, token hay dữ liệu thật không?
- Ticket chỉ được tạo sau xác nhận rõ chưa?
- Tool result error nào cần review thủ công?

## B7. Technical reflection

- Fix nào thuộc `system_prompt.md`?
- Fix nào thuộc `tools.yaml`?
- Failure nào không thể chỉ nhìn automatic score?
- Nếu có thêm một vòng, nhóm sẽ thử hypothesis nào?

# PHẦN C — Checkout trước khi nộp

Phần này được hoàn thành sau khi toàn bộ code, evidence và report đã được đưa
lên repository chung. Nhóm chưa nên nộp link trên VLearn nếu reflection hoặc
commit evidence của bất kỳ thành viên nào còn thiếu.

## C1. Reflection chung của nhóm

Các thành viên thảo luận và viết một reflection chung. Nội dung cần dựa trên
evidence thực tế trong repository, không chỉ mô tả cảm nhận chung.

- Mục tiêu nào của nhóm đã hoàn thành? Dẫn đến artifact hoặc run tương ứng.
- Hypothesis hoặc thay đổi nào tạo ra cải thiện rõ nhất?
- Failure quan trọng nào vẫn chưa xử lý được hoàn toàn?
- Nhóm đã phân chia, review và tích hợp công việc như thế nào?
- Nếu có thêm một vòng, nhóm sẽ ưu tiên thay đổi và kiểm chứng điều gì?

**Reflection chung của nhóm:**

> Viết reflection tại đây và dẫn link/path đến evidence liên quan.

## C2. Self-reflection của từng thành viên

Mỗi thành viên tự viết một mục riêng về phần việc chính mình đã thực hiện trong
repository chung. Không viết thay hoặc gộp nhiều thành viên vào một câu trả lời.
Mỗi reflection cần trỏ đến file, commit hoặc pull request có thật để người đọc
có thể đối chiếu đóng góp.

Sao chép mẫu dưới đây cho từng thành viên:

### Họ tên — MSSV

- **Vai trò/phần việc được nhận:**
- **Những gì tôi đã thay đổi trong repo chung:**
- **File hoặc artifact liên quan:**
- **Commit hash hoặc pull request:**
- **Một quyết định kỹ thuật tôi đã đưa ra và lý do:**
- **Khó khăn tôi gặp và cách tôi xử lý:**
- **Điều tôi học được từ phần việc này:**
- **Nếu làm lại, tôi sẽ cải thiện điều gì:**

Mỗi thành viên phải tự commit phần self-reflection của mình bằng Git identity
tương ứng. Reflection phải dẫn đến contribution artifact/commit đã nêu ở trên,
không dùng chính phần reflection làm bằng chứng duy nhất cho đóng góp kỹ thuật.

## C3. Final checkout

Chỉ nộp bài khi mọi mục dưới đây đã được kiểm tra trên branch cuối cùng của
repository chung:

- [ ] `TEAMMATES.md` có đủ họ tên, MSSV, GitHub username và vai trò.
- [ ] Mỗi thành viên có ít nhất một commit trong lịch sử branch nộp bài.
- [ ] Phần reflection chung của nhóm đã hoàn thành và có evidence.
- [ ] Mỗi thành viên đã tự viết và commit self-reflection của mình.
- [ ] `system_prompt.md`, `tools.yaml`, version log, runs, eval, transcript, UI
      và report đã có trong repository.
- [ ] Không có `.env`, API key, token, dữ liệu thật, cache hoặc generated ticket.
- [ ] Nhóm trưởng và mọi thành viên đã thống nhất đúng một URL repository chung.
- [ ] Nhóm trưởng và mọi thành viên sẽ nộp cùng URL đó trên VLearn.

**URL repository chung dùng để nộp:**

> URL:
