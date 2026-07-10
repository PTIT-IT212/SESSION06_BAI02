# BÀI 2: Thực hành Xây dựng Thuật toán Khuyến mãi Phức tạp


## 1. Tầm quan trọng của thứ tự ưu tiên tính toán

Trong nghiệp vụ kế toán, tài chính và thương mại điện tử, thứ tự áp dụng giảm giá, thuế và phí vận chuyển ảnh hưởng trực tiếp đến số tiền cuối cùng. Nếu áp dụng coupon trước giảm giá sản phẩm, hoặc tính VAT trước khi trừ chiết khấu, hệ thống có thể tính sai thuế, gây thất thoát doanh thu hoặc làm khách hàng bị tính phí không công bằng. Vì vậy prompt cần yêu cầu AI phân tích từng bước, nêu công thức rõ ràng và dry-run bằng dữ liệu cụ thể trước khi sinh mã.

## 2. Prompt CoT được thiết kế

```text
Bạn là Senior Backend Engineer phụ trách module thanh toán của SpeedyCart.

Hãy giải quyết bài toán tính tổng tiền đơn hàng bằng cách suy luận theo từng bước rõ ràng. Không bỏ qua thứ tự ưu tiên nghiệp vụ.

Bối cảnh:
Hệ thống cần tính tổng tiền cuối cùng khi checkout. Các quy tắc gồm:
1. Tính tổng tiền gốc của các mặt hàng.
2. Áp dụng giảm giá trực tiếp theo sản phẩm.
3. Áp dụng coupon: giảm 10% tổng đơn nhưng tối đa 100,000 VND, chỉ áp dụng nếu tổng sau giảm giá sản phẩm đạt tối thiểu 500,000 VND. Trong test case cụ thể, coupon là giảm cố định 50,000 VND với điều kiện đơn tối thiểu 500,000 VND.
4. Áp dụng loyalty discount: Gold giảm 5%, Platinum giảm 10% trên tổng tiền đã trừ coupon.
5. Cộng VAT 10% trên tổng tiền sau tất cả chiết khấu.
6. Cộng shipping fee: 30,000 VND cho dưới 10km, mỗi km tiếp theo cộng 5,000 VND. Miễn phí nếu tổng sau chiết khấu và trước thuế lớn hơn 1,000,000 VND.

Yêu cầu:
- Phân tích thứ tự đúng của các bước tính toán.
- Liệt kê công thức từng bước.
- Dry-run bằng test case:
  + 2 sản phẩm A, giá gốc 400,000 VND/sản phẩm, giảm trực tiếp 10%.
  + 1 sản phẩm B, giá gốc 300,000 VND, không giảm.
  + Coupon giảm 50,000 VND, điều kiện tối thiểu 500,000 VND.
  + Khách hàng hạng Gold giảm 5%.
  + Khoảng cách giao hàng 12km.
- Sau khi phân tích, sinh class Java `OrderCalculator` có method `calculateTotal(Order order)`.
- Bắt buộc dùng `BigDecimal` cho tiền tệ, không dùng double/float.
- Code cần rõ ràng, dễ test, có enum membership level và model tối thiểu cần thiết.

Format đầu ra:
1. Phân tích logic theo bước.
2. Công thức.
3. Dry-run chi tiết.
4. Java code hoàn chỉnh.
```

## 3. Minh chứng chạy thực tế - Text log phản hồi của AI

```markdown
# Phân tích thuật toán tính tổng tiền đơn hàng

## 1. Thứ tự tính toán đúng

Thứ tự hợp lý là:

1. Tính tổng tiền gốc của toàn bộ sản phẩm.
2. Trừ giảm giá trực tiếp trên từng sản phẩm.
3. Kiểm tra điều kiện và áp dụng coupon trên tổng sau giảm giá sản phẩm.
4. Áp dụng loyalty discount trên tổng sau coupon.
5. Tính VAT 10% trên tổng sau tất cả chiết khấu.
6. Tính shipping fee dựa vào khoảng cách, nhưng miễn phí nếu tổng sau chiết khấu trước thuế lớn hơn 1,000,000 VND.
7. Tổng cuối cùng = subtotal sau chiết khấu + VAT + shipping fee.

## 2. Công thức

Gọi:

- `originalSubtotal = sum(unitPrice * quantity)`
- `productDiscountAmount = sum(unitPrice * discountRate * quantity)`
- `subtotalAfterProductDiscount = originalSubtotal - productDiscountAmount`
- `couponDiscount = eligible ? min(configCoupon, maxCoupon) : 0`
- `subtotalAfterCoupon = subtotalAfterProductDiscount - couponDiscount`
- `loyaltyDiscount = subtotalAfterCoupon * loyaltyRate`
- `subtotalAfterAllDiscounts = subtotalAfterCoupon - loyaltyDiscount`
- `vat = subtotalAfterAllDiscounts * 10%`
- `shippingFee = 0 nếu subtotalAfterAllDiscounts > 1,000,000, ngược lại 30,000 + max(0, distanceKm - 10) * 5,000`
- `finalTotal = subtotalAfterAllDiscounts + vat + shippingFee`

## 3. Dry-run test case

### Dữ liệu đầu vào

- Sản phẩm A: 2 sản phẩm, 400,000 VND/sản phẩm, giảm trực tiếp 10%.
- Sản phẩm B: 1 sản phẩm, 300,000 VND, không giảm.
- Coupon: giảm 50,000 VND nếu đơn tối thiểu 500,000 VND.
- Membership: Gold, giảm 5%.
- Khoảng cách: 12km.

### Bước 1: Tổng tiền gốc

A = 2 × 400,000 = 800,000 VND  
B = 1 × 300,000 = 300,000 VND  
Tổng gốc = 1,100,000 VND

### Bước 2: Giảm giá sản phẩm

A giảm 10%: 800,000 × 10% = 80,000 VND  
B không giảm = 0 VND  
Tổng sau giảm giá sản phẩm = 1,100,000 - 80,000 = 1,020,000 VND

### Bước 3: Coupon

Điều kiện tối thiểu 500,000 VND được thỏa vì 1,020,000 >= 500,000.  
Coupon giảm = 50,000 VND  
Tổng sau coupon = 1,020,000 - 50,000 = 970,000 VND

### Bước 4: Loyalty discount

Gold giảm 5%.  
Loyalty discount = 970,000 × 5% = 48,500 VND  
Tổng sau tất cả chiết khấu = 970,000 - 48,500 = 921,500 VND

### Bước 5: VAT

VAT = 921,500 × 10% = 92,150 VND

### Bước 6: Shipping fee

Tổng sau chiết khấu trước thuế = 921,500 VND, không vượt 1,000,000 VND nên không được miễn phí ship.  
Khoảng cách 12km: 30,000 + (12 - 10) × 5,000 = 40,000 VND

### Bước 7: Tổng cuối cùng

Final total = 921,500 + 92,150 + 40,000 = 1,053,650 VND

## 4. Java OrderCalculator

```java
package com.speedycart.checkout;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.List;
import java.util.Objects;

public class OrderCalculator {

    private static final BigDecimal VAT_RATE = new BigDecimal("0.10");
    private static final BigDecimal GOLD_RATE = new BigDecimal("0.05");
    private static final BigDecimal PLATINUM_RATE = new BigDecimal("0.10");
    private static final BigDecimal FREE_SHIPPING_THRESHOLD = new BigDecimal("1000000");
    private static final BigDecimal DEFAULT_SHIPPING_FEE = new BigDecimal("30000");
    private static final BigDecimal EXTRA_KM_FEE = new BigDecimal("5000");
    private static final BigDecimal COUPON_MINIMUM_ORDER = new BigDecimal("500000");

    public CalculationResult calculateTotal(Order order) {
        validate(order);

        BigDecimal originalSubtotal = calculateOriginalSubtotal(order.getItems());
        BigDecimal subtotalAfterProductDiscount = calculateSubtotalAfterProductDiscount(order.getItems());
        BigDecimal couponDiscount = calculateCouponDiscount(order, subtotalAfterProductDiscount);
        BigDecimal subtotalAfterCoupon = subtotalAfterProductDiscount.subtract(couponDiscount);
        BigDecimal loyaltyDiscount = calculateLoyaltyDiscount(subtotalAfterCoupon, order.getMembershipLevel());
        BigDecimal subtotalAfterAllDiscounts = subtotalAfterCoupon.subtract(loyaltyDiscount);
        BigDecimal vat = money(subtotalAfterAllDiscounts.multiply(VAT_RATE));
        BigDecimal shippingFee = calculateShippingFee(subtotalAfterAllDiscounts, order.getDistanceKm());
        BigDecimal finalTotal = subtotalAfterAllDiscounts.add(vat).add(shippingFee);

        return new CalculationResult(
                money(originalSubtotal),
                money(subtotalAfterProductDiscount),
                money(couponDiscount),
                money(loyaltyDiscount),
                money(subtotalAfterAllDiscounts),
                money(vat),
                money(shippingFee),
                money(finalTotal)
        );
    }

    private BigDecimal calculateOriginalSubtotal(List<OrderItem> items) {
        return items.stream()
                .map(item -> item.getUnitPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    private BigDecimal calculateSubtotalAfterProductDiscount(List<OrderItem> items) {
        return items.stream()
                .map(item -> {
                    BigDecimal lineTotal = item.getUnitPrice().multiply(BigDecimal.valueOf(item.getQuantity()));
                    BigDecimal discount = lineTotal.multiply(item.getProductDiscountRate());
                    return lineTotal.subtract(discount);
                })
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    private BigDecimal calculateCouponDiscount(Order order, BigDecimal subtotalAfterProductDiscount) {
        if (order.getCouponAmount() == null) {
            return BigDecimal.ZERO;
        }
        if (subtotalAfterProductDiscount.compareTo(COUPON_MINIMUM_ORDER) < 0) {
            return BigDecimal.ZERO;
        }
        return order.getCouponAmount().min(subtotalAfterProductDiscount);
    }

    private BigDecimal calculateLoyaltyDiscount(BigDecimal subtotalAfterCoupon, MembershipLevel level) {
        BigDecimal rate = switch (level) {
            case GOLD -> GOLD_RATE;
            case PLATINUM -> PLATINUM_RATE;
            default -> BigDecimal.ZERO;
        };
        return money(subtotalAfterCoupon.multiply(rate));
    }

    private BigDecimal calculateShippingFee(BigDecimal subtotalAfterAllDiscounts, int distanceKm) {
        if (subtotalAfterAllDiscounts.compareTo(FREE_SHIPPING_THRESHOLD) > 0) {
            return BigDecimal.ZERO;
        }
        int extraKm = Math.max(0, distanceKm - 10);
        return DEFAULT_SHIPPING_FEE.add(EXTRA_KM_FEE.multiply(BigDecimal.valueOf(extraKm)));
    }

    private void validate(Order order) {
        if (order == null || order.getItems() == null || order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order/items must not be null or empty");
        }
        for (OrderItem item : order.getItems()) {
            if (item.getUnitPrice() == null || item.getUnitPrice().compareTo(BigDecimal.ZERO) < 0) {
                throw new IllegalArgumentException("Invalid unit price");
            }
            if (item.getQuantity() <= 0) {
                throw new IllegalArgumentException("Quantity must be positive");
            }
            if (item.getProductDiscountRate() == null) {
                item.setProductDiscountRate(BigDecimal.ZERO);
            }
        }
    }

    private BigDecimal money(BigDecimal value) {
        return value.setScale(0, RoundingMode.HALF_UP);
    }

    public enum MembershipLevel {
        NORMAL, GOLD, PLATINUM
    }

    public static class Order {
        private List<OrderItem> items;
        private BigDecimal couponAmount;
        private MembershipLevel membershipLevel;
        private int distanceKm;

        public List<OrderItem> getItems() { return items; }
        public void setItems(List<OrderItem> items) { this.items = items; }
        public BigDecimal getCouponAmount() { return couponAmount; }
        public void setCouponAmount(BigDecimal couponAmount) { this.couponAmount = couponAmount; }
        public MembershipLevel getMembershipLevel() { return Objects.requireNonNullElse(membershipLevel, MembershipLevel.NORMAL); }
        public void setMembershipLevel(MembershipLevel membershipLevel) { this.membershipLevel = membershipLevel; }
        public int getDistanceKm() { return distanceKm; }
        public void setDistanceKm(int distanceKm) { this.distanceKm = distanceKm; }
    }

    public static class OrderItem {
        private String productId;
        private BigDecimal unitPrice;
        private int quantity;
        private BigDecimal productDiscountRate;

        public String getProductId() { return productId; }
        public void setProductId(String productId) { this.productId = productId; }
        public BigDecimal getUnitPrice() { return unitPrice; }
        public void setUnitPrice(BigDecimal unitPrice) { this.unitPrice = unitPrice; }
        public int getQuantity() { return quantity; }
        public void setQuantity(int quantity) { this.quantity = quantity; }
        public BigDecimal getProductDiscountRate() { return productDiscountRate; }
        public void setProductDiscountRate(BigDecimal productDiscountRate) { this.productDiscountRate = productDiscountRate; }
    }

    public record CalculationResult(
            BigDecimal originalSubtotal,
            BigDecimal subtotalAfterProductDiscount,
            BigDecimal couponDiscount,
            BigDecimal loyaltyDiscount,
            BigDecimal subtotalAfterAllDiscounts,
            BigDecimal vat,
            BigDecimal shippingFee,
            BigDecimal finalTotal
    ) {}
}
```
```
