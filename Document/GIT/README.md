## Hướng dẫn sử dụng GIT command

Hướng thao tác với git bằng giao diện dòng lệnh

```bash

    # Ubuntu/Debian
    sudo apt-get update
    sudo apt-get install git


    # Cài GIT cho Centos nếu chưa có\
    sudo yum install git

    # Trên macOS
    brew install git
```

```bash
    #Xem các lệnh hiện có
    git --help
```

Clone source về
```bash
git clone https://gitlab.com/kubernetes9104623/kubernetesyamlfile.git
```


```bash
    #Liệt kê tất cả các nhánh hiện có:
    git branch

    # Chuyển nhánh
    git checkout tên_nhánh

    # Tạo nhánh mới và chuyển sang nhánh đó
    git checkout -b tên_nhánh_mới

    # Merge từ một nhánh khác vào (Tương tự với merge nhánh cá nhân vào 1 nhánh khác.)
    git merge tên_nhánh_nguồn

```

```bash

    # Cập nhật các thay đổi
    git pull

    # Thêm (add) và cam kết (commit) các thay đổi đã được giải quyết
    git add .

    # Commit
    git commit -m "Resolve merge conflicts"

    # Đẩy lên
    git push
```


## Tất cả các lệnh
## Git Commands

##### git add
Thêm các tệp vào khu vực đệm (staging area) để chuẩn bị cho lần cam kết (commit) tiếp theo.  
**Ví dụ:** `git add file.txt` (thêm tệp `file.txt` vào khu vực đệm).

#### git bisect
Dùng để tìm kiếm cam kết gây ra một lỗi bằng cách sử dụng tìm kiếm nhị phân.  
**Ví dụ:** `git bisect start`, `git bisect bad`, `git bisect good` (bắt đầu quá trình tìm kiếm lỗi bằng tìm kiếm nhị phân).

#### git branch
Liệt kê, tạo hoặc xóa các nhánh (branch).  
**Ví dụ:** `git branch` (liệt kê các nhánh), `git branch new-branch` (tạo nhánh mới tên `new-branch`).

#### git checkout
Chuyển đổi giữa các nhánh hoặc khôi phục các tệp từ cây làm việc (working tree) hoặc khu vực đệm.  
**Ví dụ:** `git checkout new-branch` (chuyển sang nhánh `new-branch`), `git checkout file.txt` (khôi phục tệp `file.txt` từ khu vực đệm).

#### git clone
Tạo một bản sao của kho lưu trữ (repository) từ một kho lưu trữ gốc.  
**Ví dụ:** `git clone https://github.com/user/repo.git` (clone kho lưu trữ từ GitHub).

#### git commit
Ghi lại các thay đổi vào kho lưu trữ.  
**Ví dụ:** `git commit -m "Commit message"` (cam kết các thay đổi với lời nhắn "Commit message").

#### git diff
Hiển thị các thay đổi giữa các cam kết, cam kết và cây làm việc, v.v.  
**Ví dụ:** `git diff` (hiển thị các thay đổi giữa cây làm việc và khu vực đệm), `git diff HEAD` (hiển thị các thay đổi giữa cây làm việc và cam kết gần nhất).

#### git fetch
Tải về các đối tượng (objects) và tham chiếu (refs) từ một kho lưu trữ khác.  
**Ví dụ:** `git fetch origin` (tải về các đối tượng và tham chiếu từ kho lưu trữ gốc `origin`).

#### git grep
Tìm kiếm và in ra các dòng khớp với một mẫu trong cây làm việc.  
**Ví dụ:** `git grep "pattern" *.txt` (tìm kiếm chuỗi "pattern" trong các tệp `.txt`).

#### git init
Tạo một kho lưu trữ Git mới hoặc khởi tạo lại một kho lưu trữ hiện có.  
**Ví dụ:** `git init` (khởi tạo kho lưu trữ Git mới trong thư mục hiện tại).

#### git log
Hiển thị lịch sử cam kết.  
**Ví dụ:** `git log` (hiển thị lịch sử cam kết), `git log --oneline` (hiển thị lịch sử cam kết trong dạng ngắn gọn).

#### git merge
Kết hợp hai hoặc nhiều lịch sử phát triển lại với nhau.  
**Ví dụ:** `git merge branch-name` (merge nhánh `branch-name` vào nhánh hiện tại).

#### git mv
Di chuyển hoặc đổi tên một tệp, thư mục hoặc liên kết symbolic.  
**Ví dụ:** `git mv file.txt new-file.txt` (đổi tên tệp `file.txt` thành `new-file.txt`).

#### git pull
Tải về và merge từ một kho lưu trữ khác hoặc từ một nhánh cục bộ.  
**Ví dụ:** `git pull origin master` (tải về và merge nhánh `master` từ kho lưu trữ gốc `origin`).

#### git push
Cập nhật các tham chiếu remote cùng với các đối tượng liên quan.  
**Ví dụ:** `git push origin master` (đẩy nhánh `master` lên kho lưu trữ gốc `origin`).

#### git rebase
Di chuyển các cam kết cục bộ đến đầu mới của nhánh upstream.  
**Ví dụ:** `git rebase master` (di chuyển các cam kết cục bộ đến đầu mới của nhánh `master`).

#### git reset
Đặt lại HEAD hiện tại về trạng thái được chỉ định.  
**Ví dụ:** `git reset HEAD~1` (đặt lại HEAD về cam kết trước đó), `git reset --hard` (đặt lại HEAD và cây làm việc về trạng thái được chỉ định).

#### git rm
Xóa các tệp khỏi cây làm việc và khỏi khu vực đệm.  
**Ví dụ:** `git rm file.txt` (xóa tệp `file.txt` khỏi cây làm việc và khu vực đệm).

#### git show
Hiển thị các loại đối tượng (objects) khác nhau.  
**Ví dụ:** `git show commit-hash` (hiển thị thông tin về cam kết với mã hash `commit-hash`).

#### git status
Hiển thị trạng thái của cây làm việc.  
**Ví dụ:** `git status` (hiển thị trạng thái của cây làm việc, bao gồm các tệp đã thay đổi, chưa được theo dõi, v.v.).

#### git tag
Tạo, liệt kê, xóa hoặc xác minh một đối tượng tag được ký bằng GPG.  
**Ví dụ:** `git tag v1.0.0` (tạo một tag mới với tên `v1.0.0`), `git tag -l` (liệt kê tất cả các tag).
