# UTSNikenServices
Nama : Niken Noviardiana  
Nim : 2101550017  
# Deskripsi Project
Tujuan project ini yaitu untuk memenuhi tugas project ujian tengah semester 7.
# Alat yang dibutuhkan
1. XAMPP (atau server web lain dengan PHP dan MySQL)
2. Text editor (misalnya Visual Studio Code, Notepad++, dll)
3. Postman
# Cara Instalasi dan Penggunaan
## 1. Persiapan lingkungan
a. Instal XAMPP jika belum ada.  
b. Buat folder baru bernama services di dalam direktori htdocs XAMPP Anda.
## 2. Membuat database
a. Buka phpMyAdmin (http://localhost/phpmyadmin)  
b. Buat database baru bernama services  
c. Pilih database services, lalu buka tab SQL  
d. Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:
```sql
CREATE TABLE services (
  id int(11) NOT NULL,
  name varchar(100) NOT NULL,
  duration time NOT NULL,
  price int(15) NOT NULL,
  category varchar(100) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
INSERT INTO services (id, name, duration, price, category) VALUES
(NULL, 'nail art polos', '01:30:00', '35000', 'kuku'),
(NULL, 'nail art motif 4', '01:30:00', '50000', 'kuku'),
(NULL, 'eyelash', '00:30:00', '75000', 'mata'),
(NULL, 'lashlift', '00:30:00', '40000', 'mata');
```
## 3. Membuat File PHP untuk Web Services
1. Buka text editor Anda.
2. Buat file baru dan simpan sebagai services.php di dalam folder services.
3. Salin dan tempel kode berikut ke dalam services.php:
```php
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, durationization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'], '/'));
}

function getConnection()
{
    $host = 'localhost';
    $db   = 'services';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL)
{
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM services WHERE id = ?");
            $stmt->execute([$id]);
            $services = $stmt->fetch();
            if ($services) {
                response(200, $services);
            } else {
                response(404, ["message" => "services not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM services");
            $servicess = $stmt->fetchAll();
            response(200, $servicess);
        }
        break;

    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->duration) || !isset($data->price) || !isset($data->category)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO services (name, duration, price, category) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->duration, $data->price, $data->category])) {
            response(201, ["message" => "services created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create services"]);
        }
        break;

    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "services ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->duration) || !isset($data->price) || !isset($data->category)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE services SET name = ?, duration = ?, price = ?, category = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->duration, $data->price, $data->category, $id])) {
            response(200, ["message" => "services updated"]);
        } else {
            response(500, ["message" => "Failed to update services"]);
        }
        break;

    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "services ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM services WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "services deleted"]);
        } else {
            response(500, ["message" => "Failed to delete services"]);
        }
        break;

    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
```
## 4. Pengujian dengan Postman
1. Buka Postman
2. Buat request baru untuk setiap operasi berikut:
### 1. GET All Services
- Method: GET  
- URL: http://localhost/services/services.php  
- Klik "Send"
### 2. GET Specific Services
- Method: GET  
- URL: http://localhost/services/services.php/3  
- Klik "Send"
### 3. POST New Services
- Method: POST  
- URL: http://localhost/services/services.php  
- Headers:  
  - Key: Content-Type  
  - Value: application/json  
- Body:  
  - Pilih "raw" dan "JSON"  
  - Masukkan :
```php
 {
        "name": "nail art 6 motif",
        "duration": "01:30:00",
        "price": 100000,
        "category": "kuku"
    }
```
- Klik "Send"
### 4. PUT (Update) Services
- Method: PUT  
- URL: http://localhost/services/services.php/4  
- Headers:  
  - Key: Content-Type  
  - Value: application/json  
- Body:  
  - Pilih "raw" dan "JSON"  
  - Masukkan:
```php
{
        "name": "lashlift",
        "duration": "00:30:00",
        "price": 45000,
        "category": "mata"
    }
```
- Klik "Send"
### 5. DELETE Services
- Method: DELETE  
- URL: http://localhost/services/services.php/1
- Klik "Send"
