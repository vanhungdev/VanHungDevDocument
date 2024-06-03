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
