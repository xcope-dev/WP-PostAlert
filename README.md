# WP-PostAlert
Emailowe Powiadomienia o Nowych Wpisach dla Użytkowników WordPress

# Opis projektu
en kod umożliwia automatyczne wysyłanie powiadomień e-mail do zarejestrowanych użytkowników strony WordPress po opublikowaniu nowego wpisu na blogu. Powiadomienia są wysyłane tylko do zalogowanych użytkowników o wybranych rolach. Kod dostosowuje adres nadawcy oraz nazwę nadawcy e-maili, a same wiadomości są formułowane w HTML z linkiem prowadzącym do najnowszego wpisu.

## Instalacja

1. **Skopiuj kod**: Skopiuj poniższy kod do pliku `functions.php` w motywie potomnym Twojej witryny WordPress.
2. **Dostosuj kod**:
   - Zaktualizuj adres e-mail nadawcy w funkcji `custom_wp_mail_from`.
   - Ustaw nazwę nadawcy w funkcji `custom_wp_mail_from_name`.

3. **Testuj działanie**: Po opublikowaniu nowego wpisu użytkownicy powinni otrzymać powiadomienie e-mail.

## Wymagania

- WordPress
- Motyw potomny
- Użytkownicy z odpowiednimi rolami (subscriber, author, employee)

## Uwagi

Kod umieść **w pliku `functions.php` motywu potomnego**. W ten sposób zapewniasz, że funkcja pozostanie aktywna nawet po aktualizacji głównego motywu.


### Kod

```php

/**
 * Powiadomienie email o nowym wpisie dla użytkowników
 */

// Funkcja zmieniająca adres email nadawcy
function custom_wp_mail_from($original_email_address) {
    return 'email@twojadomena.com'; // Tutaj podaj swój adres email nadawcy
}

// Funkcja zmieniająca nazwę nadawcy
function custom_wp_mail_from_name($original_email_from) {
    return 'TWOJA NAZWA'; // Podaj nazwę nadawcy
}

// Dodanie filtrów do zmiany nadawcy
add_filter('wp_mail_from', 'custom_wp_mail_from');
add_filter('wp_mail_from_name', 'custom_wp_mail_from_name');

// Funkcja wysyłająca email do zalogowanych użytkowników po opublikowaniu posta
function send_email_on_post_publish($new_status, $old_status, $post) {
    // Sprawdzenie, czy post jest nowo opublikowany (czyli nie był wcześniej opublikowany) i czy jest typu 'post'
    if ($old_status != 'publish' && $new_status == 'publish' && $post->post_type == 'post') {
        
        // Pobranie użytkowników o rolach "Subscriber", "Author" oraz "Employee"
        $args = array(
            'role__in' => array('subscriber', 'author', 'employee'), // Można dodać więcej ról w tablicy
        );
        
        // Pobierz użytkowników
        $users = get_users($args);

        // Generowanie tematu i treści emaila
        $post_title = get_the_title($post->ID);
        $subject = 'Nowy wpis na stronie: ' . $post_title;
        
        // Treść emaila z linkiem stałym i formatowaniem HTML
        $message = '<strong>Cześć!</strong><br><br>';
        $message .= 'Właśnie pojawiła się nowa publikacja: ' . $post_title . "<br><br>";
        $message .= 'Odwiedź witrynę i sprawdź <a href="#">TUTAJ</a>.'; // Podaj URL swojej witryny 

        // Ustawienie nagłówków, aby e-mail był wysyłany jako HTML
        $headers = array('Content-Type: text/html; charset=UTF-8');
        
        // Przesyłanie emaila do każdego użytkownika
        foreach ($users as $user) {
            wp_mail($user->user_email, $subject, $message, $headers);
        }
    }
}

// Podłącz funkcję do działania przy zmianie statusu posta
add_action('transition_post_status', 'send_email_on_post_publish', 10, 3);



