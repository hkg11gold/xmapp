package test_xampp;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.util.List;
import java.util.Scanner;
public class RMIClient {
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.getRegistry("localhost", 1099);         
            RMIServerInterface stub = (RMIServerInterface) registry.lookup("RMIServer");// 
            displayMenu();
            Scanner scanner = new Scanner(System.in);
            int choice = scanner.nextInt();           
            switch (choice) {// Xử lý lựa chọn của người dùng
                case 1:                    
                    addProduct(stub, scanner);// Thêm sản phẩm
                   break;
                case 2:                    
                    updateProduct(stub, scanner);// Sửa sản phẩm
                    break;
                case 3:                    
                    deleteProduct(stub, scanner);// Xóa sản phẩm
                    break;
                case 4:                    
                    searchById(stub, scanner);// Tìm kiếm sản phẩm theo ID
                    break;
                default:
                    System.out.println("Lựa chọn không hợp lệ!");
            }
        } catch (Exception e) {
            System.err.println("Client exception: " + e.toString());
            e.printStackTrace();
        }}
    private static void displayMenu() {
        System.out.println("------ Menu ------");
        System.out.println("1. Thêm sản phẩm");
        System.out.println("2. Sửa sản phẩm");
        System.out.println("3. Xóa sản phẩm");
        System.out.println("4. Tìm kiếm sản phẩm theo ID");
        System.out.println("------------------");
        System.out.print("Nhập lựa chọn của bạn: ");
    }
    private static void addProduct(RMIServerInterface stub, Scanner scanner) {
        try {
            System.out.print("Nhập tên sản phẩm: ");
            String name = scanner.next();
            System.out.print("Nhập giá sản phẩm: ");
            double price = scanner.nextDouble();
            System.out.print("Nhập số lượng trong kho: ");
            int countInStock = scanner.nextInt();
            boolean result = stub.insertData(name, price, countInStock);
            if (result) {
                System.out.println("Sản phẩm đã được thêm thành công.");
            } else {
                System.out.println("Đã xảy ra lỗi khi thêm sản phẩm.");
            }
        } catch (Exception e) {
            System.err.println("Lỗi khi thêm sản phẩm: " + e.getMessage());
        }}
    private static void updateProduct(RMIServerInterface stub, Scanner scanner) {
        try {
            System.out.print("Nhập ID của sản phẩm cần sửa: ");
            int id = scanner.nextInt();
            System.out.print("Nhập tên sản phẩm mới: ");
            String name = scanner.next();
            System.out.print("Nhập giá sản phẩm mới: ");
            double price = scanner.nextDouble();
            System.out.print("Nhập số lượng trong kho mới: ");
            int countInStock = scanner.nextInt();
            boolean result = stub.updateData(id, name, price, countInStock);
            if (result) {
                System.out.println("Sản phẩm đã được cập nhật thành công.");
            } else {
                System.out.println("Đã xảy ra lỗi khi cập nhật sản phẩm.");
            }
        } catch (Exception e) {
            System.err.println("Lỗi khi cập nhật sản phẩm: " + e.getMessage());
        }}
    private static void deleteProduct(RMIServerInterface stub, Scanner scanner) {
        try {
            System.out.print("Nhập ID của sản phẩm cần xóa: ");
            int id = scanner.nextInt();
            boolean result = stub.deleteData(id);
            if (result) {
                System.out.println("Sản phẩm đã được xóa thành công.");
            } else {
                System.out.println("Đã xảy ra lỗi khi xóa sản phẩm.");
            }
        } catch (Exception e) {
            System.err.println("Lỗi khi xóa sản phẩm: " + e.getMessage());
        }}
    private static void searchById(RMIServerInterface stub, Scanner scanner) {
        try {
            System.out.print("Nhập ID của sản phẩm cần tìm kiếm: ");
            int id = scanner.nextInt();
            List<String> result = stub.searchById(id);
            if (result != null && !result.isEmpty()) {
                System.out.println("Thông tin sản phẩm:");
                for (String item : result) {
                    System.out.println(item);
                }
            } else {
                System.out.println("Không tìm thấy sản phẩm có ID là " + id);
            }
        } catch (Exception e) {
            System.err.println("Lỗi khi tìm kiếm sản phẩm: " + e.getMessage());}}}
------------------------------------------------------------------------------------------------------
package test_xampp;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
public class RMIServer {
    public static void main(String[] args) {
        try {
            RMIServerImpl obj = new RMIServerImpl();// Tạo một đối tượng của RMIServerImpl
            Registry registry = LocateRegistry.createRegistry(1099);// Đăng ký đối tượng với RMI Registry
            registry.rebind("RMIServer", obj);
            System.out.println("Server đã sẵn sàng...");
        } catch (Exception e) {
            System.err.println("Server exception: " + e.toString());
            e.printStackTrace(); }}}
------------------------------------------------------------------------------------------------------------------------------------------
package test_xampp;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
public class RMIServerImpl extends UnicastRemoteObject implements RMIServerInterface {
    public RMIServerImpl() throws RemoteException {
        super();}
    @Override
    public boolean insertData(String name, double price, int countInStock) throws RemoteException {
        try {
            Connection conn = SQLConnection.getConnection();
            PreparedStatement statement = conn.prepareStatement("INSERT INTO product (name, price, countinstock) VALUES (?, ?, ?)");
            statement.setString(1, name);
            statement.setDouble(2, price);
            statement.setInt(3, countInStock);
            int rowsInserted = statement.executeUpdate();
            conn.close();
            return rowsInserted > 0;
        } catch (SQLException e) {
            e.printStackTrace();
            return false;}}
    @Override
    public boolean updateData(int id, String name, double price, int countInStock) throws RemoteException {
        try {
            Connection conn = SQLConnection.getConnection();
            PreparedStatement statement = conn.prepareStatement("UPDATE product SET name=?, price=?, countinstock=? WHERE id=?");
            statement.setString(1, name);
            statement.setDouble(2, price);
            statement.setInt(3, countInStock);
            statement.setInt(4, id);
            int rowsUpdated = statement.executeUpdate();
            conn.close();
            return rowsUpdated > 0;
        } catch (SQLException e) {
            e.printStackTrace();
            return false;}}
    @Override
    public boolean deleteData(int id) throws RemoteException {
        try {
            Connection conn = SQLConnection.getConnection();
            PreparedStatement statement = conn.prepareStatement("DELETE FROM product WHERE id=?");
            statement.setInt(1, id);
            int rowsDeleted = statement.executeUpdate();
            conn.close();
            return rowsDeleted > 0;
        } catch (SQLException e) {
            e.printStackTrace();
            return false;}}
    @Override
    public List<String> searchById(int id) throws RemoteException {
        List<String> data = new ArrayList<>();
        try {
            Connection conn = SQLConnection.getConnection();
            PreparedStatement statement = conn.prepareStatement("SELECT * FROM product WHERE id=?");
            statement.setInt(1, id);
            ResultSet rs = statement.executeQuery();
            while (rs.next()) {
                data.add(rs.getString("name") + " - " + rs.getInt("price"));
            }
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return data;}}
-------------------------------------------------------------------------------------------------------------------------
package test_xampp;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;
public interface RMIServerInterface extends Remote {
	boolean insertData(String name, double price, int countInStock) throws RemoteException;
	boolean updateData(int id, String name, double price, int countInStock) throws RemoteException;
	boolean deleteData(int id) throws RemoteException;
	List<String> searchById(int id) throws RemoteException;}
------------------------------------------------------------------------------------------------
package test_xampp;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
public class SQLConnection {
    public static Connection getConnection() throws SQLException {
        String url = "jdbc:mysql://localhost:3306/bai3"; // Thay đổi "bai3" nếu bạn đặt tên khác cho cơ sở dữ liệu
        String username = "root"; // Thay đổi tên đăng nhập nếu cần
        String password = ""; // Thay đổi mật khẩu nếu bạn đã thiết lập mật khẩu cho MySQL
        return DriverManager.getConnection(url, username, password);}}




