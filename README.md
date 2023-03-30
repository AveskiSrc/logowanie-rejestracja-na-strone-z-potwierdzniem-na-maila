# logowanie-rejestracja-na-strone-z-potwierdzniem-na-maila
Kod napisany w php do rejestracji , logowania oraz automatycznego potwierdzenia na maila uwaga kod wymaga MySql oraz biblioteke PHPMailer

Pełny kod 

<?php
    if(isset($_POST['register'])) {
        // połączenie z bazą danych
        $conn = mysqli_connect('localhost', 'username', 'password', 'database');

        // pobranie danych z formularza
        $username = $_POST['username'];
        $password = $_POST['password'];
        $email = $_POST['email'];

        // sprawdzenie, czy użytkownik o podanym adresie e-mail już istnieje
        $query = "SELECT * FROM users WHERE email='$email'";
        $result = mysqli_query($conn, $query);
        if(mysqli_num_rows($result) > 0) {
            echo "Użytkownik o podanym adresie e-mail już istnieje!";
        } else {
            // wygenerowanie klucza aktywacyjnego
            $activation_key = md5($email.time());

            // zapisanie danych użytkownika do bazy danych
            $query = "INSERT INTO users (username, password, email, activation_key) VALUES ('$username', '$password', '$email', '$activation_key')";
            mysqli_query($conn, $query);

            // wysłanie e-maila z kluczem aktywacyjnym
            require_once "phpmailer/PHPMailer.php";
            require_once "phpmailer/SMTP.php";

            $mail = new PHPMailer();
            $mail->IsSMTP();
            $mail->Mailer = "smtp";
            $mail->SMTPDebug  = 0;
            $mail->SMTPAuth   = true;
            $mail->SMTPSecure = "ssl";
            $mail->Host       = "smtp.gmail.com";
            $mail->Port       = 465;
            $mail->Username   = "your.email@gmail.com";
            $mail->Password   = "your.email.password";
            $mail->SetFrom('your.email@gmail.com', 'Your Name');
            $mail->AddReplyTo("your.email@gmail.com","Your Name");
            $mail->AddAddress($email);
            $mail->Subject = "Potwierdzenie rejestracji";
            $mail->MsgHTML("Witaj $username,<br><br>
                            Dziękujemy za rejestrację w naszym serwisie. Aby aktywować swoje konto, kliknij w poniższy link:<br>
                            <a href='http://www.yourwebsite.com/activate.php?key=$activation_key'>http://www.yourwebsite.com/activate.php?key=$activation_key</a>");
            if(!$mail->Send()) {
                echo "Wysyłka e-maila nie powiodła się: " . $mail->ErrorInfo;
            } else {
                echo "Rejestracja zakończona pomyślnie. Na podany adres e-mail została wysłana wiadomość z instrukcjami dotyczącymi aktywacji konta.";
            }
        }
    }
?>

<form method="post">
    <label>Nazwa użytkownika:</label><br>
    <input type="text" name="username"><br>
    <label>Has
