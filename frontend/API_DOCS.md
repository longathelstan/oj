# Thống kê các endpoint của VNOJ (backend Django)

Báo cáo sau tổng hợp các endpoint (route) do hệ thống VNOJ (fork từ DMOJ) cung cấp. Bảng liệt kê được chia theo nhóm chức năng chính như xác thực, người dùng, bài tập, nộp bài, cuộc thi, tổ chức, v.v. Mỗi endpoint bao gồm: phương thức HTTP, đường dẫn (pattern), tham số đầu vào (query/body/path), dữ liệu trả về (HTML hoặc JSON), và thông tin xác thực (có yêu cầu đăng nhập hay không). Các tuyến được dùng trong giao diện gốc (template HTML) được ghi chú để tham khảo.

## Đăng nhập / Tài khoản (Auth)

Các endpoint liên quan đến quản lý phiên người dùng:

| Phương thức | Đường dẫn                     | Tham số đầu vào                                                | Đầu ra (kết quả)                                                               | Yêu cầu xác thực         |
| :---------- | :--------------------------- | :------------------------------------------------------------- | :----------------------------------------------------------------------------- | :----------------------- |
| GET, POST   | `/accounts/login/`           | **body:** `username`, `password` (và `next` nếu redirect)      | Trang đăng nhập (HTML). Khi `POST` hợp lệ sẽ redirect tới trang chỉ định.      | Không (dùng session)     |
| GET         | `/accounts/logout/`          | --                                                             | Thực hiện logout và redirect (thường về trang chủ).                            | Có (phiên hợp lệ)        |
| GET, POST   | `/accounts/register/`        | **body:** `full_name`, `username`, `email`, `password1`, `password2`, `timezone`, `default_language`, `affiliations`... | Trang đăng ký tài khoản (HTML). `POST` tạo tài khoản mới, sau đó redirect.      | Không                    |
| GET, POST   | `/accounts/password/change/` | **body:** mật khẩu cũ, mật khẩu mới, xác nhận mật khẩu mới     | Trang đổi mật khẩu (nếu có). Cần đăng nhập.                                    | Có                       |
| GET, POST   | `/accounts/password/reset/`  | **body:** email hoặc username để reset mật khẩu                | Trang yêu cầu reset mật khẩu (nếu hỗ trợ).                                     | Không                    |

## Người dùng (User)

Các endpoint hiển thị thông tin và hoạt động liên quan đến người dùng:

| Phương thức | Đường dẫn                       | Tham số đầu vào               | Đầu ra (kết quả)                                                              | Yêu cầu xác thực |
| :---------- | :----------------------------- | :---------------------------- | :---------------------------------------------------------------------------- | :--------------- |
| GET         | `/users/`                      | `?page=<N>` (trang)           | Danh sách người dùng (Leaderboard) theo điểm. HTML hiển thị bảng xếp hạng chung. | Không            |
| GET         | `/contributors/`               | `?page=<N>` (trang)           | Danh sách các *contributor* (người đóng góp). HTML hiển thị bảng xếp hạng đóng góp. | Không            |
| GET         | `/organizations/`              | --                            | Danh sách tổ chức (organization) cùng điểm và số thành viên. HTML.           | Không            |
| GET         | `/user/<username>/`            | Path: `username`              | Trang hồ sơ người dùng (Profile). Hiển thị thông tin cơ bản, điểm, biểu tượng, v.v. | Không            |
| GET         | `/user/<username>/solved/`     | Path: `username`              | Thống kê chi tiết: danh sách bài đã giải (có điểm) của người dùng. HTML hoặc JSON. | Không            |
| GET         | `/user/<username>/blog/`       | Path: `username`              | Danh sách bài đăng (blog) của người dùng. HTML liệt kê các bài viết đã đăng. | Không            |
| GET         | `/user/<username>/statistics/` | Path: `username`              | (Nếu có) Thống kê thêm về người dùng, ví dụ lịch sử tính điểm, biểu đồ, v.v. HTML. | Không            |

## Bài tập (Problems)

| Phương thức | Đường dẫn                               | Tham số đầu vào                                                              | Đầu ra (kết quả)                                                                    | Yêu cầu xác thực     |
| :---------- | :------------------------------------- | :--------------------------------------------------------------------------- | :---------------------------------------------------------------------------------- | :------------------ |
| GET         | `/problems/`                           | `?page=<N>`, các filter (tìm kiếm, loại, category, has_editorial,...)        | Trang danh sách bài tập (HTML) với phân trang. Có form tìm kiếm và bộ lọc.          | Không                |
| GET         | `/problem/<problem_code>/`             | Path: `problem_code`                                                         | Trang chi tiết một bài tập. Hiển thị đề bài, giới hạn, điểm, nút nộp, link đến submisisons, v.v. | Không                |
| GET, POST   | `/problem/<problem_code>/submit/`      | **body:** mã nguồn (`source_code`), ngôn ngữ (`language`), `[contest_id]` (nếu nộp trong contest) | Trang nộp bài (form) và xử lý submission. `GET` hiển thị form, `POST` gửi code và redirect về submission vừa tạo. | Có (đăng nhập)      |
| GET         | `/problem/<problem_code>/submissions/` | Path: `problem_code`, `?page=<N>`, filter (trạng thái, ngôn ngữ, user, org) | Danh sách tất cả các submission cho bài này (HTML). Có phân trang và bộ lọc.       | Không                |
| GET         | `/problem/<problem_code>/rank/`        | Path: `problem_code`                                                         | Bảng xếp hạng (Best solutions) của bài này.                                         | Không                |
| GET         | `/problem/<problem_code>/statement/`   | Path: `problem_code`                                                         | Tải xuống đề bài (có thể là PDF) nếu có.                                           | Không                |

## Nộp bài (Submissions)

| Phương thức | Đường dẫn                       | Tham số đầu vào                                              | Đầu ra (kết quả)                                                             | Yêu cầu xác thực              |
| :---------- | :----------------------------- | :----------------------------------------------------------- | :--------------------------------------------------------------------------- | :--------------------------- |
| GET         | `/submissions/`                | `?page=<N>`, filter (trạng thái, ngôn ngữ, problem, contest, org, user, ...) | Trang liệt kê các submission toàn hệ thống (HTML). Có phân trang và bộ lọc.  | Không                         |
| GET         | `/submissions/<page>/`         | Path: `page` (số trang)                                      | Tương tự `/submissions/?page=<page>`.                                        | Không                         |
| GET         | `/submission/<submission_id>/` | Path: `submission_id`                                        | Trang chi tiết một submission: gồm mã nguồn, kết quả chấm, thời gian chạy, memory, v.v. (HTML). | Có (xem submission của mình hoặc có quyền) |

## Cuộc thi (Contests)

| Phương thức | Đường dẫn                               | Tham số đầu vào              | Đầu ra (kết quả)                                | Yêu cầu xác thực       |
| :---------- | :------------------------------------- | :--------------------------- | :-------------------------------------------- | :-------------------- |
| GET         | `/contests/`                           | `?page=<N>` (trang), filter (Upcoming/Past, private,...) | Danh sách các cuộc thi (Upcoming & Past). HTML hiển thị thông tin. | Không                  |
| GET         | `/contests/YYYY/M/`                    | Path: `YYYY`, `M` (tháng)    | Lịch tổ chức contest theo tháng. HTML.         | Không                  |
| GET         | `/contests.ics`                        | --                           | File ICS (Calendar).                          | Không                  |
| GET         | `/contest/<contest_slug>/`             | Path: `contest_slug`         | Trang thông tin cuộc thi (Info).              | Không (trừ private)    |
| GET         | `/contest/<contest_slug>/statistics/`  | Path: `contest_slug`         | Thống kê chung contest (nếu có).              | Có thể yêu cầu đăng nhập |
| GET         | `/contest/<contest_slug>/ranking/`     | Path: `contest_slug`         | Bảng xếp hạng contest.                        | Không                  |
| GET         | `/contest/<contest_slug>/submissions/` | Path: `contest_slug`, `?page=<N>`, filters | Danh sách submission trong contest.             | Có thể yêu cầu đăng nhập |

## Tổ chức (Organizations)

| Phương thức | Đường dẫn                             | Tham số đầu vào | Đầu ra (kết quả)                                          | Yêu cầu xác thực   |
| :---------- | :----------------------------------- | :-------------- | :-------------------------------------------------------- | :---------------- |
| GET         | `/organizations/`                    | --              | Danh sách tất cả tổ chức, kèm điểm và số thành viên. HTML. | Không              |
| GET         | `/organization/<org_id>/`            | Path: `org_id`  | Trang chính của tổ chức.                                 | Không (công khai) |
| GET         | `/organization/<org_id>/users/`      | Path: `org_id`  | Danh sách thành viên tổ chức và xếp hạng. HTML.          | Không              |
| GET         | `/organization/<org_id>/problems/`   | Path: `org_id`  | Các bài tập thuộc tổ chức. HTML.                         | Có (thành viên)   |
| GET         | `/organization/<org_id>/contests/`   | Path: `org_id`  | Các cuộc thi thuộc tổ chức. HTML.                        | Có (thành viên)   |
| GET         | `/organization/<org_id>/submissions/` | Path: `org_id`  | Submission của tổ chức. HTML.                            | Có (thành viên)   |

## Trang chủ và trang khác

| Phương thức | Đường dẫn            | Tham số đầu vào                | Đầu ra (kết quả)                                            | Yêu cầu xác thực |
| :---------- | :------------------ | :----------------------------- | :---------------------------------------------------------- | :--------------- |
| GET         | `/`                 | `?show_all_blogs=[true/false]` | Trang chủ (newsfeed). Hiển thị tin tức, top users, contributors, blogs. | Không            |
| GET         | `/custom_checkers/` | --                             | Trang hướng dẫn viết custom checker.                        | Không            |
| GET         | `/status/`          | --                             | Trang trạng thái judge (nếu có).                            | Không            |
| GET         | `/rss/`, `/atom/`  | --                             | RSS/Atom feed.                                              | Không            |