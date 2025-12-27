# AJAX w WordPress i WooCommerce - Kompletny Poradnik dla Początkujących

## Spis Treści
1. [Wstęp](#wstęp)
2. [Czym jest AJAX?](#czym-jest-ajax)
3. [AJAX w WordPress](#ajax-w-wordpress)
4. [Praktyczne Przykłady](#praktyczne-przykłady)
5. [WooCommerce + AJAX](#woocommerce--ajax)
6. [Najczęstsze Błędy](#najczęstsze-błędy)
7. [Best Practices](#best-practices)

---

## Wstęp

AJAX (Asynchronous JavaScript and XML) to technologia pozwalająca na wymianę danych z serwerem **bez przeładowania strony**. W WordPress i WooCommerce jest to niezwykle ważne dla poprawy doświadczenia użytkownika.

Ten poradnik zawiera praktyczne przykłady dla początkujących developerów.

---

## Czym jest AJAX?

### Tradycyjny Model
```
Użytkownik → Klik → Request → Przeładowanie strony → Response
```

### Model AJAX
```
Użytkownik → Klik → Request (w tle) → Response (bez przeładowania)
```

### Zalety AJAX
✅ Szybsze doświadczenie użytkownika  
✅ Brak przeładowania strony  
✅ Asynchroniczne przetwarzanie  
✅ Lepsza wydajność  

---

## AJAX w WordPress

### 1. Podstawowa Struktura

Każdy AJAX request w WordPress wymaga:
- **Frontend**: JavaScript który wysyła request
- **Backend**: Funkcja PHP (action hook) która przetwarza request

### 2. Najważniejszy Koncept: `wp_localize_script()`

Ta funkcja przekazuje dane PHP do JavaScriptu:

```php
<?php
wp_localize_script( 'script-handle', 'object-name', array(
    'ajax_url' => admin_url( 'admin-ajax.php' ),
    'nonce'    => wp_create_nonce( 'action-name' )
) );
```

### Co to jest Nonce?

**Nonce** (number used once) to token bezpieczeństwa zapobiegający nieautoryzowanym żądaniom AJAX.

---

## Praktyczne Przykłady

### Przykład 1: Proste Licznik Kliknięć

#### Krok 1: Rejestracja Skryptu (functions.php)

```php
<?php
function moja_wtyczka_enqueue_scripts() {
    // Rejestruj skrypt
    wp_register_script(
        'moja-ajax-script',
        get_template_directory_uri() . '/js/ajax.js',
        array( 'jquery' ),
        '1.0.0',
        true
    );
    
    // Wczytaj skrypt
    wp_enqueue_script( 'moja-ajax-script' );
    
    // Przekaż zmienne do JavaScriptu
    wp_localize_script( 'moja-ajax-script', 'moja_ajax', array(
        'ajax_url' => admin_url( 'admin-ajax.php' ),
        'nonce'    => wp_create_nonce( 'licznik_nonce' )
    ) );
}
add_action( 'wp_enqueue_scripts', 'moja_wtyczka_enqueue_scripts' );
```

#### Krok 2: JavaScript (js/ajax.js)

```javascript
jQuery(document).ready(function($) {
    $('#licznik-button').on('click', function(e) {
        e.preventDefault();
        
        $.ajax({
            url: moja_ajax.ajax_url,
            type: 'POST',
            data: {
                action: 'licznik_action',
                nonce: moja_ajax.nonce
            },
            success: function(response) {
                $('#licznik-wynik').text('Kliknięcia: ' + response);
                console.log('Sukces!', response);
            },
            error: function(error) {
                console.log('Błąd:', error);
                alert('Coś poszło nie tak!');
            }
        });
    });
});
```

#### Krok 3: PHP Handler (functions.php)

```php
<?php
function licznik_ajax_handler() {
    // Weryfikuj nonce
    check_ajax_referer( 'licznik_nonce', 'nonce' );
    
    // Pobierz obecną wartość z opcji
    $licznik = get_option( 'licznik_klikniecia', 0 );
    
    // Zwiększ o 1
    $licznik++;
    
    // Zapisz nową wartość
    update_option( 'licznik_klikniecia', $licznik );
    
    // Wyślij odpowiedź JSON
    wp_send_json_success( $licznik );
    
    // Ważne: wp_send_json_success() kończy script
}
add_action( 'wp_ajax_licznik_action', 'licznik_ajax_handler' );
add_action( 'wp_ajax_nopriv_licznik_action', 'licznik_ajax_handler' );
```

#### HTML Szablonu

```html
<button id="licznik-button" class="btn btn-primary">Kliknij mnie</button>
<div id="licznik-wynik">Kliknięcia: 0</div>
```

---

### Przykład 2: Formularz z Walidacją

#### JavaScript

```javascript
jQuery(document).ready(function($) {
    $('#kontakt-form').on('submit', function(e) {
        e.preventDefault();
        
        var formData = {
            action: 'wyslij_kontakt',
            nonce: moja_ajax.nonce,
            imie: $('#imie').val(),
            email: $('#email').val(),
            wiadomosc: $('#wiadomosc').val()
        };
        
        // Zablokuj przycisk podczas wysyłania
        var $button = $(this).find('button[type="submit"]');
        $button.prop('disabled', true).text('Wysyłam...');
        
        $.ajax({
            url: moja_ajax.ajax_url,
            type: 'POST',
            data: formData,
            success: function(response) {
                if(response.success) {
                    alert('Wiadomość wysłana!');
                    $('#kontakt-form')[0].reset();
                } else {
                    alert('Błąd: ' + response.data.message);
                }
                $button.prop('disabled', false).text('Wyślij');
            },
            error: function() {
                alert('Błąd serwera!');
                $button.prop('disabled', false).text('Wyślij');
            }
        });
    });
});
```

#### PHP Handler

```php
<?php
function wyslij_kontakt_handler() {
    check_ajax_referer( 'licznik_nonce', 'nonce' );
    
    // Sanityzacja danych
    $imie = sanitize_text_field( $_POST['imie'] );
    $email = sanitize_email( $_POST['email'] );
    $wiadomosc = sanitize_textarea_field( $_POST['wiadomosc'] );
    
    // Walidacja
    if( empty( $imie ) || empty( $email ) || empty( $wiadomosc ) ) {
        wp_send_json_error( array(
            'message' => 'Wszystkie pola są wymagane!'
        ) );
    }
    
    if( !is_email( $email ) ) {
        wp_send_json_error( array(
            'message' => 'Podaj prawidłowy email!'
        ) );
    }
    
    // Wyślij email
    $admin_email = get_option( 'admin_email' );
    $subject = 'Nowa wiadomość z formularza kontaktowego';
    $body = "Imię: {$imie}\nEmail: {$email}\n\nWiadomość:\n{$wiadomosc}";
    
    wp_mail( $admin_email, $subject, $body );
    
    // Wyślij odpowiedź sukcesu
    wp_send_json_success( array(
        'message' => 'Wiadomość została wysłana!'
    ) );
}
add_action( 'wp_ajax_wyslij_kontakt', 'wyslij_kontakt_handler' );
add_action( 'wp_ajax_nopriv_wyslij_kontakt', 'wyslij_kontakt_handler' );
```

---

### Przykład 3: Dynamiczne Ładowanie Postów

#### JavaScript

```javascript
jQuery(document).ready(function($) {
    var page = 1;
    
    $('#load-more-btn').on('click', function() {
        page++;
        
        $.ajax({
            url: moja_ajax.ajax_url,
            type: 'POST',
            data: {
                action: 'laduj_posty',
                nonce: moja_ajax.nonce,
                page: page,
                per_page: 5
            },
            success: function(response) {
                if(response.success) {
                    // Dodaj nowe posty do kontenera
                    $('#posty-container').append(response.data.html);
                    
                    // Ukryj przycisk jeśli nie ma więcej postów
                    if(!response.data.has_more) {
                        $('#load-more-btn').hide();
                    }
                } else {
                    alert('Błąd przy ładowaniu postów');
                }
            }
        });
    });
});
```

#### PHP Handler

```php
<?php
function laduj_posty_handler() {
    check_ajax_referer( 'licznik_nonce', 'nonce' );
    
    $page = intval( $_POST['page'] );
    $per_page = intval( $_POST['per_page'] );
    
    $args = array(
        'posts_per_page' => $per_page,
        'paged'          => $page,
        'post_type'      => 'post',
        'post_status'    => 'publish'
    );
    
    $query = new WP_Query( $args );
    
    // Sprawdź czy są jeszcze posty
    $has_more = $page < $query->max_num_pages;
    
    ob_start();
    
    if( $query->have_posts() ) {
        while( $query->have_posts() ) {
            $query->the_post();
            ?>
            <article class="post">
                <h3><?php the_title(); ?></h3>
                <p><?php the_excerpt(); ?></p>
                <a href="<?php the_permalink(); ?>">Czytaj więcej</a>
            </article>
            <?php
        }
        wp_reset_postdata();
    }
    
    $html = ob_get_clean();
    
    wp_send_json_success( array(
        'html'     => $html,
        'has_more' => $has_more
    ) );
}
add_action( 'wp_ajax_laduj_posty', 'laduj_posty_handler' );
add_action( 'wp_ajax_nopriv_laduj_posty', 'laduj_posty_handler' );
```

---

## WooCommerce + AJAX

### Przykład 1: AJAX w Koszyku

```javascript
jQuery(document).ready(function($) {
    // Aktualizacja ilości produktu w koszyku
    $('body').on('change', '.woocommerce-cart-form input.qty', function() {
        var $form = $(this).closest('form');
        var cart_hash = JSON.stringify($form.serializeArray());
        
        $.ajax({
            url: moja_ajax.ajax_url,
            type: 'POST',
            data: {
                action: 'update_cart_ajax',
                nonce: moja_ajax.nonce,
                form_data: $form.serializeArray()
            },
            success: function(response) {
                if(response.success) {
                    $('body').trigger('updated_woo_ajax');
                }
            }
        });
    });
});
```

### Przykład 2: Dodawanie Produktu do Koszyka bez Przeładowania

```javascript
jQuery(document).ready(function($) {
    $('body').on('click', '.single_add_to_cart_button', function(e) {
        // Dla prostych produktów bez zmiennych
        if($(this).hasClass('ajax_add_to_cart')) {
            // WooCommerce już obsługuje to natively
            return;
        }
        
        // Dla produktów ze zmiennymi
        e.preventDefault();
        
        var $form = $(this).closest('form');
        var formData = $form.serialize();
        
        $.ajax({
            url: moja_ajax.ajax_url,
            type: 'POST',
            data: {
                action: 'woo_add_to_cart',
                nonce: moja_ajax.nonce,
                product_id: $('[name="product_id"]').val(),
                product_data: formData
            },
            success: function(response) {
                if(response.success) {
                    alert('Produkt dodany do koszyka!');
                    $('body').trigger('wc_fragment_refresh');
                } else {
                    alert('Błąd: ' + response.data);
                }
            }
        });
    });
});
```

### Przykład 3: Filtrowanie Produktów

```javascript
jQuery(document).ready(function($) {
    // Przy zmianie filtra
    $('.product-filter select').on('change', function() {
        var filters = {};
        
        $('.product-filter select').each(function() {
            var filter_name = $(this).attr('name');
            var filter_value = $(this).val();
            
            filters[filter_name] = filter_value;
        });
        
        $.ajax({
            url: moja_ajax.ajax_url,
            type: 'POST',
            data: {
                action: 'filter_products',
                nonce: moja_ajax.nonce,
                filters: filters
            },
            success: function(response) {
                if(response.success) {
                    $('.products').html(response.data.html);
                }
            },
            beforeSend: function() {
                $('.products').html('<p>Ładowanie...</p>');
            }
        });
    });
});
```

#### PHP Handler dla Filtrowania

```php
<?php
function filter_products_handler() {
    check_ajax_referer( 'licznik_nonce', 'nonce' );
    
    $filters = isset( $_POST['filters'] ) ? $_POST['filters'] : array();
    
    $args = array(
        'post_type'      => 'product',
        'posts_per_page' => 12,
        'tax_query'      => array()
    );
    
    // Dodaj filtry do query
    if( !empty( $filters['category'] ) ) {
        $args['tax_query'][] = array(
            'taxonomy' => 'product_cat',
            'field'    => 'term_id',
            'terms'    => $filters['category']
        );
    }
    
    // Meta query dla atrybutów
    $args['meta_query'] = array();
    
    if( !empty( $filters['price_range'] ) ) {
        list( $min, $max ) = explode( '-', $filters['price_range'] );
        $args['meta_query'][] = array(
            'key'     => '_price',
            'value'   => array( $min, $max ),
            'compare' => 'BETWEEN',
            'type'    => 'NUMERIC'
        );
    }
    
    $query = new WP_Query( $args );
    
    ob_start();
    woocommerce_product_loop_start();
    
    if( $query->have_posts() ) {
        while( $query->have_posts() ) {
            $query->the_post();
            wc_get_template_part( 'content', 'product' );
        }
    } else {
        echo '<p>Brak produktów spełniających kryteria.</p>';
    }
    
    woocommerce_product_loop_end();
    wp_reset_postdata();
    
    $html = ob_get_clean();
    
    wp_send_json_success( array(
        'html' => $html
    ) );
}
add_action( 'wp_ajax_filter_products', 'filter_products_handler' );
add_action( 'wp_ajax_nopriv_filter_products', 'filter_products_handler' );
```

---

## Najczęstsze Błędy

### ❌ Błąd 1: Brak Nonce Weryfikacji

**Zle:**
```php
function moja_akcja() {
    $dane = $_POST['dane'];
    // Brak check_ajax_referer()
}
```

**Dobrze:**
```php
function moja_akcja() {
    check_ajax_referer( 'moja_akcja_nonce', 'nonce' );
    $dane = sanitize_text_field( $_POST['dane'] );
}
```

---

### ❌ Błąd 2: Brak Sanityzacji Danych

**Zle:**
```php
$imie = $_POST['imie'];
update_option( 'moje_imie', $imie ); // Niebezpieczne!
```

**Dobrze:**
```php
$imie = sanitize_text_field( $_POST['imie'] );
update_option( 'moje_imie', $imie );
```

---

### ❌ Błąd 3: Nie Usando wp_send_json_*

**Zle:**
```php
echo json_encode( array( 'success' => true ) );
die();
```

**Dobrze:**
```php
wp_send_json_success( array( 'success' => true ) );
```

---

### ❌ Błąd 4: Zapominanie o `wp_ajax_nopriv_`

Jeśli AJAX ma być dostępny dla niezalogowanych użytkowników:

```php
// Tylko zalogowani
add_action( 'wp_ajax_moja_akcja', 'handler_function' );

// Zalogowani + niezalogowani
add_action( 'wp_ajax_moja_akcja', 'handler_function' );
add_action( 'wp_ajax_nopriv_moja_akcja', 'handler_function' );
```

---

### ❌ Błąd 5: Problem z `wp_localize_script()` - Wrong Order

**Zle:**
```php
wp_enqueue_script( 'moj-skrypt', ... );
wp_localize_script( 'moj-skrypt', ... ); // Skrypt jeszcze nie jest wczytany
wp_register_script( 'moj-skrypt', ... ); // To jest po localize!
```

**Dobrze:**
```php
wp_register_script( 'moj-skrypt', ... );
wp_enqueue_script( 'moj-skrypt' );
wp_localize_script( 'moj-skrypt', ... ); // Zawsze OSTATNIA
```

---

## Best Practices

### 1. Zawsze Używaj Nonce

```php
// Frontend
data: {
    nonce: moja_ajax.nonce
}

// Backend
check_ajax_referer( 'action-nonce', 'nonce' );
```

### 2. Sanityzuj Wszystkie Dane Input

```php
$text = sanitize_text_field( $_POST['text'] );
$email = sanitize_email( $_POST['email'] );
$url = esc_url( $_POST['url'] );
$html = wp_kses_post( $_POST['html'] );
```

### 3. Dodaj Error Handling

```javascript
$.ajax({
    // ...
    error: function(xhr, status, error) {
        console.error('AJAX Error:', error);
        console.error('Status:', status);
        console.error('Response:', xhr.responseText);
    }
});
```

### 4. Użyj `wp_send_json_success()` i `wp_send_json_error()`

```php
// Sukces
wp_send_json_success( array( 'message' => 'OK' ) );

// Błąd
wp_send_json_error( array( 'message' => 'Coś poszło nie tak' ), 400 );
```

### 5. Limituj Częstotliwość Requestów

```javascript
var is_loading = false;

$('#load-more').on('click', function() {
    if( is_loading ) return;
    
    is_loading = true;
    
    $.ajax({
        // ...
        complete: function() {
            is_loading = false;
        }
    });
});
```

### 6. Dodaj Loading Indicator

```javascript
$.ajax({
    beforeSend: function() {
        $('.loader').show();
    },
    complete: function() {
        $('.loader').hide();
    }
});
```

### 7. Waliduj po Stronie Serwera

```php
// Zawsze waliduj dane na serwerze, nie polegaj na walidacji frontendu
if( strlen( $imie ) < 2 ) {
    wp_send_json_error( array(
        'message' => 'Imię musi mieć co najmniej 2 znaki'
    ) );
}
```

---

## Czek-Lista dla Początkujących

- ✅ Czy używam `wp_localize_script()` aby przekazać `ajax_url` i `nonce`?
- ✅ Czy weryfikuję nonce z `check_ajax_referer()`?
- ✅ Czy sanityzuję wszystkie dane z `$_POST` i `$_GET`?
- ✅ Czy używam `wp_send_json_success()` lub `wp_send_json_error()`?
- ✅ Czy mam hook dla zalogowanych (`wp_ajax_`) i niezalogowanych (`wp_ajax_nopriv_`)?
- ✅ Czy obsługuję błędy w JavaScript (error callback)?
- ✅ Czy dodałem loading indicator?
- ✅ Czy przetestuję AJAX w konsoli przeglądarki?

---

## Przydatne Funkcje WordPress

```php
// Sanityzacja
sanitize_text_field()      // Tekst
sanitize_email()           // Email
esc_url()                  // URL
wp_kses_post()             // HTML
intval()                   // Liczba całkowita
floatval()                 // Liczba zmiennoprzecinkowa

// Weryfikacja
is_email()                 // Czy email jest prawidłowy
is_array()                 // Czy tablica
is_string()                // Czy string

// AJAX Response
wp_send_json_success()     // Sukces
wp_send_json_error()       // Błąd
wp_send_json()             // Niestandardowa odpowiedź

// Nonce
wp_create_nonce()          // Utwórz nonce
wp_verify_nonce()          // Weryfikuj nonce
check_ajax_referer()       // Weryfikuj nonce w AJAX
```

---

## Przykłady Kodu do Pobrania

Wszystkie przykłady z tego poradnika znajdziesz w sekcji "Praktyczne Przykłady".

### Szybki Start Template

```php
<?php
// Rejestracja skryptu
function moja_wtyczka_init() {
    wp_register_script(
        'moja-ajax',
        get_template_directory_uri() . '/js/ajax.js',
        array( 'jquery' ),
        '1.0.0',
        true
    );
    wp_enqueue_script( 'moja-ajax' );
    wp_localize_script( 'moja-ajax', 'moja_ajax', array(
        'ajax_url' => admin_url( 'admin-ajax.php' ),
        'nonce'    => wp_create_nonce( 'moja_akcja_nonce' )
    ) );
}
add_action( 'wp_enqueue_scripts', 'moja_wtyczka_init' );

// AJAX Handler
function moja_akcja_handler() {
    check_ajax_referer( 'moja_akcja_nonce', 'nonce' );
    
    // Twój kod tutaj
    
    wp_send_json_success( array( 'message' => 'OK' ) );
}
add_action( 'wp_ajax_moja_akcja', 'moja_akcja_handler' );
add_action( 'wp_ajax_nopriv_moja_akcja', 'moja_akcja_handler' );
```

---

## Podsumowanie

AJAX to potężne narzędzie dla WordPress i WooCommerce. Kluczowe punkty:

1. **Zawsze zabezpiecz** swój kod za pomocą nonce
2. **Zawsze sanityzuj** dane wejściowe
3. **Zawsze obsługuj** błędy
4. **Zawsze testuj** w konsoli przeglądarki
5. **Zawsze dokumentuj** swój kod

Powodzenia w nauce AJAX! 🚀

---

**Ostatnia aktualizacja**: 2025-12-27

Pytania? Sprawdź oficjalną dokumentację:
- [WordPress AJAX](https://developer.wordpress.org/plugins/javascript/ajax/)
- [WooCommerce AJAX Hooks](https://woocommerce.com/document/hooks/)
