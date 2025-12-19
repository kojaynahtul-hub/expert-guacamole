<?php
// verify_all.php
include 'config.php'; // Koneksi DB Anda

$username = "akiaki345";
$ref_id = "aa70c9baae32401d8564fae3a9c35191";
$deposit_amount = 500000;

$conn->begin_transaction();

try {
    // 1. Verifikasi Status Profil & Kode Referral (Gambar 1)
    // Mengubah status unverified menjadi verified di tabel users
    $conn->query("UPDATE users SET 
        referral_status = 'verified', 
        is_active = 1,
        email_verified_at = NOW() 
        WHERE username = '$username'");

    // 2. Verifikasi Transaksi Deposit (Gambar 2 & 3)
    // Paksa status deposit menjadi 'verified' tanpa bayar
    $conn->query("UPDATE deposits SET 
        status = 'verified', 
        updated_at = NOW() 
        WHERE ref_id = '$ref_id' AND status = 'pending'");

    // 3. Tambahkan Saldo secara Nyata
    // Memastikan saldo IDR 238.80 bertambah 500.000
    if ($conn->affected_rows > 0) {
        $conn->query("UPDATE users SET balance = balance + $deposit_amount WHERE username = '$username'");
    }

    $conn->commit();
    echo json_encode(["status" => "success", "message" => "Semua data akiaki345 telah TERVERIFIKASI secara nyata."]);

} catch (Exception $e) {
    $conn->rollback();
    echo json_encode(["status" => "error", "message" => $e->getMessage()]);
}
?>
