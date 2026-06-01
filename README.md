# BeautySeq Recommender Web Demo

Web demo nay minh hoa he thong goi y san pham thuong mai dien tu tren bo du lieu Amazon Beauty. Demo su dung `Popularity` cho nguoi dung moi va chuyen sang `BSARec` khi da co du chuoi hanh vi.

## Muc tieu

- Hien thi catalog san pham Beauty.
- Ghi lai hanh vi nguoi dung khi xem san pham hoac them vao gio hang.
- Tao goi y ca nhan hoa dua tren chuoi hanh vi gan nhat.
- Trinh bay ket qua danh gia offline cua cac mo hinh: Popularity, GRU4Rec, SASRec, BERT4Rec va BSARec.

## Cach chay

Chay tu thu muc `Web demo`:

```powershell
cd "C:\Study\DoAnV2\Web demo"
python server.py
```

Mo trinh duyet:

```text
http://127.0.0.1:8501
```

Tai khoan demo co san:

```text
demo / demo123
```

Co the dang ky tai khoan moi trong giao dien de tao chuoi hanh vi sach khi demo.

## Luong goi y

He thong co 2 che do:

1. `Popularity`

   Dung khi nguoi dung chua dang nhap hoac co it hon 3 hanh vi san pham. Danh sach duoc xep theo so lan xuat hien cua san pham trong `train_history.csv`.

2. `BSARec`

   Dung tu hanh vi san pham thu 3 tro di. Server lay chuoi item gan nhat cua nguoi dung, dua vao checkpoint BSARec, tinh diem cho cac item va tra ve top-K san pham co diem cao nhat.

Tim kiem va loc danh muc chi la loc catalog, khong phai mo hinh goi y.

## Hanh vi nguoi dung

Hanh vi demo duoc luu trong:

```text
demo_events.jsonl
```

Moi dong la mot JSON event, vi du:

```json
{"event_id":"evt_xxx","user_id":"demo_xxx","type":"view_product","item_id":"594","query":null,"created_at":"2026-06-01T01:02:31.856002+00:00"}
```

Nhung event duoc tinh vao chuoi hanh vi cho BSARec:

```text
view_product
add_to_cart
```

Nhung event khac co the duoc luu nhung khong dua vao chuoi BSARec:

```text
search
open_cart
remove_from_cart
```

## Cach BSARec tao goi y

Luong xu ly trong `server.py`:

1. Doc event cua user tu `demo_events.jsonl`.
2. Lay cac item co event `view_product` hoac `add_to_cart`.
3. Giu toi da 50 item gan nhat.
4. Neu chuoi co it hon 3 hanh vi, dung `Popularity`.
5. Neu chuoi co tu 3 hanh vi, dua chuoi vao BSARec.
6. Model tinh diem cho toan bo item trong khong gian embedding.
7. Loai cac item nguoi dung da xem hoac da co trong gio hang.
8. Lay top-K item diem cao nhat va tra ve cho web.

## File quan trong

```text
server.py                         Backend API va luong goi y
app.js                            Logic frontend
index.html                        Giao dien web
styles.css                        CSS giao dien
items.csv                         Thong tin co ban san pham
item_details.csv                  Mo ta, store, category, price
item_images.csv                   Anh san pham
train_history.csv                 Lich su dung de tinh popularity
metrics.csv                       Bang ket qua danh gia offline
demo_users.json                   Tai khoan demo
demo_carts.json                   Gio hang demo
demo_events.jsonl                 Log hanh vi demo
beauty_recommender_outputs/       Artifact danh gia va cau hinh mo hinh
```

Checkpoint BSARec duoc load tu:

```text
..\BSARec-main\src\output\BSARec_Beauty_best.pt
```

## Ket qua danh gia

Ket qua chinh nam trong `metrics.csv`.

| Model | HR@10 | NDCG@10 | MRR |
|---|---:|---:|---:|
| Popularity | 0.011984 | 0.005613 | 0.005655 |
| GRU4Rec | 0.030631 | 0.014568 | 0.014301 |
| SASRec | 0.048607 | 0.025990 | 0.024557 |
| BERT4Rec | 0.075661 | 0.042717 | 0.039234 |
| BSARec | 0.095023 | 0.057535 | 0.052632 |

BSARec dat ket qua cao nhat tren cac chi so HR@K, NDCG@K va MRR. Dieu nay cho thay mo hinh hoc duoc tin hieu tu chuoi hanh vi tot hon cac baseline.

## Goi y kich ban demo

Nen tao tai khoan moi de chuoi hanh vi sach, sau do click cac san pham cung chu de. Vi du nhom skin care/body care:

```text
1089  Liquid Trust
518   Estee Lauder Perfumed Body Powder
1284  Bikini Hair Removal System + Shave Cream
1111  Diva By Emanuel Ungaro Body Lotion
62    Witch Vera Gel
```

Sau khi click, quay ve trang chu. Neu banner hien `BSARec`, co the giai thich:

```text
Nguoi dung vua xem lien tiep cac san pham skin care/body care/grooming.
BSARec dung chuoi hanh vi nay de xep hang lai san pham va uu tien cac item co kha nang phu hop tiep theo.
```

## Luu y khi bao cao

- BSARec khong phai bo loc theo tu khoa hay theo danh muc.
- Model hoc mau chuoi hanh vi: sau khi nguoi dung tuong tac voi cac item A, B, C thi item nao co kha nang xuat hien tiep theo.
- Mot so goi y co the nhin khac danh muc vi du lieu Amazon Beauty co nhieu san pham nhieu va nhieu title khong sach.
- Khi demo, nen dung tai khoan moi va click cac san pham cung chu de de ket qua de giai thich hon.
