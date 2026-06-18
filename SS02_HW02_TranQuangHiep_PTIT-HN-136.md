Bạn là một chuyên gia Java và bảo mật ứng dụng.

Hãy viết một class `VNPayPaymentService` bằng Java để thực hiện thanh toán trực tuyến thông qua VNPay.

Yêu cầu bảo mật bắt buộc:

1. Tuyệt đối không hardcode API Key, Secret Key, Access Token hoặc bất kỳ thông tin nhạy cảm nào trong mã nguồn.
2. Khóa bí mật VNPay phải được nạp từ biến môi trường hệ điều hành bằng:

   ```java
   System.getenv("VNPAY_HASH_SECRET")
   ```
3. Trước khi sử dụng khóa bí mật, phải kiểm tra:

    * Biến môi trường không tồn tại (null).
    * Biến môi trường rỗng hoặc chỉ chứa khoảng trắng.
4. Nếu khóa bí mật không hợp lệ, phải ném ra ngoại lệ phù hợp (IllegalStateException) với thông báo rõ ràng để ngăn hệ thống khởi động hoặc xử lý giao dịch trong trạng thái không an toàn.
5. Thiết kế mã nguồn theo hướng production-ready:

    * Tách cấu hình thành class riêng.
    * Có phương thức lấy secret key an toàn.
    * Có phương thức tạo chữ ký HMAC SHA512 cho dữ liệu thanh toán.
    * Có phương thức tạo URL thanh toán VNPay.
6. Chỉ sử dụng dữ liệu cấu hình mẫu như URL sandbox, không sử dụng hoặc yêu cầu bất kỳ khóa bí mật thật nào.
7. Trả về mã nguồn Java hoàn chỉnh có chú thích giải thích các biện pháp bảo mật đã áp dụng.


import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;

/**
* Cấu hình VNPay.
* Không chứa bất kỳ khóa bí mật nào trong mã nguồn.
  */
  public final class VNPayConfig {

  private VNPayConfig() {
  }

  // URL Sandbox của VNPay (không phải dữ liệu nhạy cảm)
  public static final String VNP_URL =
  "https://sandbox.vnpayment.vn/paymentv2/vpcpay.html";

  /**
    * Lấy Secret Key từ biến môi trường.
    * Không hardcode trong source code.
      */
      public static String getHashSecret() {
      String secret = System.getenv("VNPAY_HASH_SECRET");

      if (secret == null || secret.trim().isEmpty()) {
      throw new IllegalStateException(
      "Missing required environment variable: VNPAY_HASH_SECRET"
      );
      }

      return secret;
      }
      }
* import javax.crypto.Mac;
  import javax.crypto.spec.SecretKeySpec;
  import java.net.URLEncoder;
  import java.nio.charset.StandardCharsets;
  import java.util.Map;
  import java.util.TreeMap;
  import java.util.stream.Collectors;

/**
* Service xử lý thanh toán VNPay.
  */
  public class VNPayPaymentService {

  /**
    * Tạo chữ ký HMAC SHA512 cho dữ liệu thanh toán.
      */
      public String hmacSHA512(String data) {
      try {
      String secretKey = VNPayConfig.getHashSecret();

           Mac hmac512 = Mac.getInstance("HmacSHA512");
           SecretKeySpec secretKeySpec =
                   new SecretKeySpec(secretKey.getBytes(StandardCharsets.UTF_8),
                           "HmacSHA512");

           hmac512.init(secretKeySpec);

           byte[] hashBytes =
                   hmac512.doFinal(data.getBytes(StandardCharsets.UTF_8));

           StringBuilder result = new StringBuilder();
           for (byte b : hashBytes) {
               result.append(String.format("%02x", b));
           }

           return result.toString();

      } catch (Exception e) {
      throw new RuntimeException(
      "Unable to generate VNPay signature", e);
      }
      }

  /**
    * Tạo URL thanh toán VNPay.
      */
      public String createPaymentUrl(Map<String, String> inputParams) {

      try {
      // Sắp xếp tham số theo yêu cầu VNPay
      Map<String, String> sortedParams =
      new TreeMap<>(inputParams);

           String queryString = sortedParams.entrySet()
                   .stream()
                   .map(entry ->
                           encode(entry.getKey()) + "=" +
                           encode(entry.getValue()))
                   .collect(Collectors.joining("&"));

           String secureHash = hmacSHA512(queryString);

           return VNPayConfig.VNP_URL
                   + "?"
                   + queryString
                   + "&vnp_SecureHash="
                   + secureHash;

      } catch (Exception e) {
      throw new RuntimeException(
      "Failed to create VNPay payment URL", e);
      }
      }

  private String encode(String value) {
  return URLEncoder.encode(value, StandardCharsets.UTF_8);
  }
  }
  Ví dụ cấu hình biến môi trường

Linux/macOS

export VNPAY_HASH_SECRET=your_secret_key_here

Windows CMD

set VNPAY_HASH_SECRET=your_secret_key_here

Windows PowerShell

$env:VNPAY_HASH_SECRET="your_secret_key_here"

Điểm cải thiện bảo mật chính:

Không gửi hoặc lưu Secret Key trong prompt.
Không hardcode Secret Key trong mã nguồn.
Sử dụng System.getenv() để nạp khóa bí mật.
Kiểm tra null và chuỗi rỗng trước khi sử dụng.
Ném IllegalStateException để ngăn hệ thống hoạt động khi thiếu cấu hình bảo mật.