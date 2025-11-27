# Service: 
- auth
- user
- product
- inventory
- order
- payment
- notification
- discount


# Database: 
- auth keycloak db
- user user_db
- product product_db
- inventory inventory_db
- order order_db
- payment payment_db
- discount discount_db
- notification cloud mongodb


# Communication:
- gRPC
- RabbitMQ
- redis

## gRPC
Order ↔ Product
Order ↔ Inventory
Order ↔ Payment
Order ↔ Discount
User ↔ Auth

## RabbitMQ
OrderCreated
StockReserved
PaymentSucceeded
DiscountLocked
OrderCompleted
OrderCancelled
DiscountExpired
VoucherGenerated
NotificationEmail/SMS/Push

## Redis Pub/Sub
Realtime notification
Websocket updates


# Cache:
- redis

## Redis Cache
Product detail/list
User profile
Inventory availability
Rate-limiting
Token/session


# Tích hợp thanh toán


# cronjob
## ✅ 1. Expire discount / voucher hết hạn

Chạy mỗi 5 phút hoặc 1 giờ:

Tự động deactivate coupon hết hạn.

Update status: active → expired.

Emit event: DiscountExpired.

Xoá cache Redis.

Use case: Voucher Black Friday, Cyber Monday, Flash Sale…

## ✅ 2. Reset usage daily/weekly/monthly

Ví dụ voucher có rule:

1 user dùng 1 lần / ngày

Tối đa 500 lần / tháng

Tổng usage reset ngày 1 mỗi tháng

Cronjob chạy:

00:00 mỗi ngày

00:00 ngày 1 mỗi tháng

## ✅ 3. Auto-activate discount theo lịch

Dùng cho marketing campaign:

Khuyến mãi 11.11

Khuyến mãi cuối tuần

Campaign sinh nhật user

Flash Sale lúc 12:00

Cronjob chạy:

WHEN current_timestamp >= discount.start_time
→ set status = active

## ✅ 4. Auto-deactivate campaign theo lịch

Ngược lại:

WHEN now > discount.end_time
→ set status = inactive


Giúp marketing không cần can thiệp thủ công.
