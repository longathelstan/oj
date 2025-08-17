# Thống kê các endpoint của VNOJ (backend Django)

Báo cáo sau tổng hợp các endpoint (route) do hệ thống VNOJ (fork từ
DMOJ) cung cấp. Bảng liệt kê được chia theo nhóm chức năng chính như xác
thực, người dùng, bài tập, nộp bài, cuộc thi, tổ chức, v.v. Mỗi endpoint
bao gồm: phương thức HTTP, đường dẫn (pattern), tham số đầu vào
(query/body/path), dữ liệu trả về (HTML hoặc JSON), và thông tin xác
thực (có yêu cầu đăng nhập hay không). Các tuyến được dùng trong giao
diện gốc (template HTML) được ghi chú để tham khảo.

## Đăng nhập / Tài khoản (Auth)

Các endpoint liên quan đến quản lý phiên người dùng:

  ----------------------------------------------------------------------------------------------------
  Phương   Đường dẫn                      Tham số đầu vào       Đầu ra (kết quả)            Yêu cầu
  thức                                                                                      xác thực
  -------- ------------------------------ --------------------- --------------------------- ----------
  GET,     `/accounts/login/`             **body:** `username`, Trang đăng nhập (HTML). Khi Không
  POST                                    `password` (và `next` `POST` hợp lệ sẽ redirect   (dùng
                                          nếu redirect)         tới trang chỉ định.         session)

  GET      `/accounts/logout/`            --                    Thực hiện logout và         Có (phiên
                                                                redirect (thường về trang   hợp lệ)
                                                                chủ).                       

  GET,     `/accounts/register/`          **body:**             Trang đăng ký tài khoản     Không
  POST                                    `full_name`,          (HTML). `POST` tạo tài      
                                          `username`, `email`,  khoản mới, sau đó redirect. 
                                          `password1`,                                      
                                          `password2`,                                      
                                          `timezone`,                                       
                                          `default_language`,                               
                                          `affiliations`...                                 

  GET,     `/accounts/password/change/`   **body:** mật khẩu    Trang đổi mật khẩu (nếu     Có
  POST                                    cũ, mật khẩu mới, xác có). Cần đăng nhập.         
                                          nhận mật khẩu mới                                 

  GET,     `/accounts/password/reset/`    **body:** email hoặc  Trang yêu cầu reset mật     Không
  POST                                    username để reset mật khẩu (nếu hỗ trợ).          
                                          khẩu                                              
  ----------------------------------------------------------------------------------------------------

## Người dùng (User)

Các endpoint hiển thị thông tin và hoạt động liên quan đến người dùng:

  ------------------------------------------------------------------------------------------------
  Phương   Đường dẫn                        Tham số đầu   Đầu ra (kết quả)                 Yêu cầu
  thức                                      vào                                            xác
                                                                                           thực
  -------- -------------------------------- ------------- -------------------------------- -------
  GET      `/users/`                        `?page=<N>`   Danh sách người dùng             Không
                                            (trang)       (Leaderboard) theo điểm. HTML    
                                                          hiển thị bảng xếp hạng chung.    

  GET      `/contributors/`                 `?page=<N>`   Danh sách các *contributor*      Không
                                            (trang)       (người đóng góp). HTML hiển thị  
                                                          bảng xếp hạng đóng góp.          

  GET      `/organizations/`                --            Danh sách tổ chức (organization) Không
                                                          cùng điểm và số thành viên.      
                                                          HTML.                            

  GET      `/user/<username>/`              Path:         Trang hồ sơ người dùng           Không
                                            `username`    (Profile). Hiển thị thông tin cơ 
                                                          bản, điểm, biểu tượng, v.v.      

  GET      `/user/<username>/solved/`       Path:         Thống kê chi tiết: danh sách bài Không
                                            `username`    đã giải (có điểm) của người      
                                                          dùng. HTML hoặc JSON.            

  GET      `/user/<username>/blog/`         Path:         Danh sách bài đăng (blog) của    Không
                                            `username`    người dùng. HTML liệt kê các bài 
                                                          viết đã đăng.                    

  GET      `/user/<username>/statistics/`   Path:         (Nếu có) Thống kê thêm về người  Không
                                            `username`    dùng, ví dụ lịch sử tính điểm,   
                                                          biểu đồ, v.v. HTML.              
  ------------------------------------------------------------------------------------------------

## Bài tập (Problems)

  ------------------------------------------------------------------------------------------------------------
  Phương   Đường dẫn                                Tham số đầu vào      Đầu ra (kết quả)              Yêu cầu
  thức                                                                                                 xác
                                                                                                       thực
  -------- ---------------------------------------- -------------------- ----------------------------- -------
  GET      `/problems/`                             `?page=<N>`, các     Trang danh sách bài tập       Không
                                                    filter (tìm kiếm,    (HTML) với phân trang. Có     
                                                    loại, category,      form tìm kiếm và bộ lọc.      
                                                    has_editorial,...)                                 

  GET      `/problem/<problem_code>/`               Path: `problem_code` Trang chi tiết một bài tập.   Không
                                                                         Hiển thị đề bài, giới hạn,    
                                                                         điểm, nút nộp, link đến       
                                                                         submisisons, v.v.             

  GET,     `/problem/<problem_code>/submit/`        **body:** mã nguồn   Trang nộp bài (form) và xử lý Có
  POST                                              (`source_code`),     submission. `GET` hiển thị    (đăng
                                                    ngôn ngữ             form, `POST` gửi code và      nhập)
                                                    (`language`),        redirect về submission vừa    
                                                    `[contest_id]` (nếu  tạo.                          
                                                    nộp trong contest)                                 

  GET      `/problem/<problem_code>/submissions/`   Path:                Danh sách tất cả các          Không
                                                    `problem_code`,      submission cho bài này        
                                                    `?page=<N>`, filter  (HTML). Có phân trang và bộ   
                                                    (trạng thái, ngôn    lọc.                          
                                                    ngữ, user, org)                                    

  GET      `/problem/<problem_code>/rank/`          Path: `problem_code` Bảng xếp hạng (Best           Không
                                                                         solutions) của bài này.       

  GET      `/problem/<problem_code>/statement/`     Path: `problem_code` Tải xuống đề bài (có thể là   Không
                                                                         PDF) nếu có.                  
  ------------------------------------------------------------------------------------------------------------

## Nộp bài (Submissions)

  -------------------------------------------------------------------------------------------------------
  Phương   Đường dẫn                        Tham số đầu vào   Đầu ra (kết quả)               Yêu cầu xác
  thức                                                                                       thực
  -------- -------------------------------- ----------------- ------------------------------ ------------
  GET      `/submissions/`                  `?page=<N>`,      Trang liệt kê các submission   Không
                                            filter (trạng     toàn hệ thống (HTML). Có phân  
                                            thái, ngôn ngữ,   trang và bộ lọc.               
                                            problem, contest,                                
                                            org, user, ...)                                  

  GET      `/submissions/<page>/`           Path: `page` (số  Tương tự                       Không
                                            trang)            `/submissions/?page=<page>`.   

  GET      `/submission/<submission_id>/`   Path:             Trang chi tiết một submission: Có (xem
                                            `submission_id`   gồm mã nguồn, kết quả chấm,    submission
                                                              thời gian chạy, memory, v.v.   của mình
                                                              (HTML).                        hoặc có
                                                                                             quyền)
  -------------------------------------------------------------------------------------------------------

## Cuộc thi (Contests)

  ---------------------------------------------------------------------------------------------------------
  Phương   Đường dẫn                                Tham số đầu vào   Đầu ra (kết quả)           Yêu cầu
  thức                                                                                           xác thực
  -------- ---------------------------------------- ----------------- -------------------------- ----------
  GET      `/contests/`                             `?page=<N>`       Danh sách các cuộc thi     Không
                                                    (trang), filter   (Upcoming & Past). HTML    
                                                    (Upcoming/Past,   hiển thị thông tin.        
                                                    private,...)                                 

  GET      `/contests/YYYY/M/`                      Path: `YYYY`, `M` Lịch tổ chức contest theo  Không
                                                    (tháng)           tháng. HTML.               

  GET      `/contests.ics`                          --                File ICS (Calendar).       Không

  GET      `/contest/<contest_slug>/`               Path:             Trang thông tin cuộc thi   Không (trừ
                                                    `contest_slug`    (Info).                    private)

  GET      `/contest/<contest_slug>/statistics/`    Path:             Thống kê chung contest     Có thể yêu
                                                    `contest_slug`    (nếu có).                  cầu đăng
                                                                                                 nhập

  GET      `/contest/<contest_slug>/ranking/`       Path:             Bảng xếp hạng contest.     Không
                                                    `contest_slug`                               

  GET      `/contest/<contest_slug>/submissions/`   Path:             Danh sách submission trong Có thể yêu
                                                    `contest_slug`,   contest.                   cầu đăng
                                                    `?page=<N>`,                                 nhập
                                                    filters                                      
  ---------------------------------------------------------------------------------------------------------

## Tổ chức (Organizations)

  ---------------------------------------------------------------------------------------------------
  Phương   Đường dẫn                               Tham số đầu Đầu ra (kết quả)              Yêu cầu
  thức                                             vào                                       xác thực
  -------- --------------------------------------- ----------- ----------------------------- --------
  GET      `/organizations/`                       --          Danh sách tất cả tổ chức, kèm Không
                                                               điểm và số thành viên. HTML.  

  GET      `/organization/<org_id>/`               Path:       Trang chính của tổ chức.      Không
                                                   `org_id`                                  (công
                                                                                             khai)

  GET      `/organization/<org_id>/users/`         Path:       Danh sách thành viên tổ chức  Không
                                                   `org_id`    và xếp hạng. HTML.            

  GET      `/organization/<org_id>/problems/`      Path:       Các bài tập thuộc tổ chức.    Có
                                                   `org_id`    HTML.                         (thành
                                                                                             viên)

  GET      `/organization/<org_id>/contests/`      Path:       Các cuộc thi thuộc tổ chức.   Có
                                                   `org_id`    HTML.                         (thành
                                                                                             viên)

  GET      `/organization/<org_id>/submissions/`   Path:       Submission của tổ chức. HTML. Có
                                                   `org_id`                                  (thành
                                                                                             viên)
  ---------------------------------------------------------------------------------------------------

## Trang chủ và trang khác

  -----------------------------------------------------------------------------------------------------
  Phương   Đường dẫn             Tham số đầu vào                  Đầu ra (kết quả)            Yêu cầu
  thức                                                                                        xác thực
  -------- --------------------- -------------------------------- --------------------------- ---------
  GET      `/`                   `?show_all_blogs=[true/false]`   Trang chủ (newsfeed). Hiển  Không
                                                                  thị tin tức, top users,     
                                                                  contributors, blogs.        

  GET      `/custom_checkers/`   --                               Trang hướng dẫn viết custom Không
                                                                  checker.                    

  GET      `/status/`            --                               Trang trạng thái judge (nếu Không
                                                                  có).                        

  GET      `/rss/`, `/atom/`     --                               RSS/Atom feed.              Không
  -----------------------------------------------------------------------------------------------------
