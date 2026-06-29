# BÀI 2: Thực hành Gỡ lỗi từ Stack Trace
## Nội dung Prompt gỡ lỗi
- Vai trò: Bạn là Java Debugger chuyên nghiệp với nhiều năm kinh nghiệm phát triển hệ thống ngân hàng sử dụng Java 17, JDBC và cơ sở dữ liệu quan hệ
- Bối cảnh: Khi chạy chức năng lưu giao dịch vào cơ sở dữ liệu, chương trình bị sập và xuất hiện lỗi DataIntegrityViolationException do vi phạm ràng buộc khóa ngoại (Foreign Key Constraint).
- Mã nguồn gây lỗi:

      import java.sql.Connection;
      import java.sql.PreparedStatement;

      public class TransactionRepository {
      
          private Connection connection;
          public void saveTransaction(String transactionId, Long userId, double amount) throws Exception {
      
              // Lỗi logic: Không kiểm tra xem userId có tồn tại trong bảng Users hay không
      
              // Dẫn đến lỗi vi phạm khóa ngoại (Foreign Key Constraint Violation) dưới CSDL
      
              String sql = "INSERT INTO transactions (id, user_id, amount) VALUES (?, ?, ?)";
      
              PreparedStatement ps = connection.prepareStatement(sql);
      
              ps.setString(1, transactionId);
      
              ps.setLong(2, userId);
      
              ps.setDouble(3, amount);
      
              ps.executeUpdate();
          }
      }


- Stack Trace

      org.h2.jdbc.JdbcSQLIntegrityConstraintViolationException:
      Referential integrity constraint violation:
      
      "CONSTRAINT_FOREIGN_KEY_USER:
      PUBLIC.TRANSACTIONS FOREIGN KEY(USER_ID)
      REFERENCES PUBLIC.USERS(ID) (99)";
      
      SQL statement:
      
      INSERT INTO transactions (id, user_id, amount)
      VALUES (?, ?, ?)
      
      [23506-214]
      
      at org.h2.message.DbException.getJdbcSQLException(DbException.java:502)
      
      at TransactionRepository.saveTransaction(TransactionRepository.java:13)
      
      at Main.main(Main.java:10)
- Nhiệm vụ:
Đóng vai trò Java Debugger chuyên nghiệp và thực hiện theo đúng thứ tự sau:
  + Bước 1:
    * Giải nghĩa từng phần của Stack Trace.
  + Bước 2:
    * Xác định nguyên nhân gốc rễ (Root Cause).
  + Bước 3:
    * Giải thích vì sao lỗi chỉ xuất hiện khi executeUpdate().
  + Bước 4:
    * Đề xuất cách khắc phục theo Best Practice.
  + Bước 5:
    * Viết lại TransactionRepository theo các yêu cầu:

          Java 17
          JDBC
          Không thay đổi tên phương thức saveTransaction()
          Trước khi INSERT phải kiểm tra userId có tồn tại trong bảng users hay không.
          Kiểm tra bằng câu lệnh:
          SELECT COUNT(*) FROM users WHERE id = ?
          Nếu user không tồn tại:
          Không thực hiện INSERT.
          Ném ngoại lệ tùy chỉnh UserNotFoundException.
          Nếu user tồn tại:
          Thực hiện INSERT bình thường.
          Sử dụng PreparedStatement.
          Sử dụng try-with-resources.
          Không để CSDL tự ném lỗi Foreign Key.
          Code rõ ràng, dễ đọc và có comment giải thích các đoạn quan trọng.
  + Bước 6:
    Sinh đầy đủ mã nguồn:
    
    UserNotFoundException.java
    TransactionRepository.java (đã sửa an toàn)
    
    Không rút gọn mã nguồn.
## Minh chứng chạy thực tế
- UserNotFoundException.java

        public class UserNotFoundException extends Exception {
      
          public UserNotFoundException(String message) {
              super(message);
          }
      }
- TransactionRepository.java

      import java.sql.Connection;
      import java.sql.PreparedStatement;
      import java.sql.ResultSet;
      
      public class TransactionRepository {
      
          private Connection connection;
      
          public TransactionRepository(Connection connection) {
              this.connection = connection;
          }
      
          public void saveTransaction(String transactionId,
                                      Long userId,
                                      double amount) throws Exception {
      
              if (!userExists(userId)) {
                  throw new UserNotFoundException(
                          "User với id = " + userId + " không tồn tại."
                  );
              }
      
              String insertSql =
                      "INSERT INTO transactions(id, user_id, amount) VALUES (?, ?, ?)";
      
              try (PreparedStatement ps =
                           connection.prepareStatement(insertSql)) {
      
                  ps.setString(1, transactionId);
                  ps.setLong(2, userId);
                  ps.setDouble(3, amount);
      
                  ps.executeUpdate();
              }
          }
      
          private boolean userExists(Long userId) throws Exception {
      
              String checkSql =
                      "SELECT COUNT(*) FROM users WHERE id = ?";
      
              try (PreparedStatement ps =
                           connection.prepareStatement(checkSql)) {
      
                  ps.setLong(1, userId);
      
                  try (ResultSet rs = ps.executeQuery()) {
      
                      if (rs.next()) {
                          return rs.getInt(1) > 0;
                      }
                  }
              }
      
              return false;
          }
      }
