# Dokumentacja Haków WooCommerce

## Spis Treści

1. [Wprowadzenie](#wprowadzenie)
2. [Typy Haków](#typy-haków)
3. [Haki Produktów](#haki-produktów)
4. [Haki Koszyka](#haki-koszyka)
5. [Haki Zamówienia](#haki-zamówienia)
6. [Haki Użytkownika](#haki-użytkownika)
7. [Haki Płatności](#haki-płatności)
8. [Haki Szablonów](#haki-szablonów)
9. [Przykłady Kodu](#przykłady-kodu)
10. [Najlepsze Praktyki](#najlepsze-praktyki)

---

## Wprowadzenie

WooCommerce to najpopularniejszy wtyczka sklepu internetowego dla WordPress'a. System haków (hooks) pozwala na rozszerzanie funkcjonalności bez modyfikacji plików rdzenia wtyczki.

Istnieją dwa główne typy haków:
- **Akcje (Actions)** - punkty, w których możesz wstawić swój kod
- **Filtry (Filters)** - punkty, w których możesz modyfikować istniejące dane

---

## Typy Haków

### Akcje (Actions)

Akcje pozwalają na dodanie kodu w określonych punktach WooCommerce.

```php
add_action( 'hook_name', 'your_function_name', priority, accepted_args );
```

**Parametry:**
- `hook_name` - nazwa haka
- `your_function_name` - funkcja do wykonania
- `priority` - priorytet wykonania (domyślnie 10, niższe wartości wykonują się wcześniej)
- `accepted_args` - liczba argumentów akceptowanych przez funkcję

### Filtry (Filters)

Filtry pozwalają na modyfikację istniejących danych.

```php
add_filter( 'filter_name', 'your_function_name', priority, accepted_args );
```

---

## Haki Produktów

### woocommerce_product_set_stock

Wywoływany, gdy zmieniany jest stan magazynowy produktu.

```php
do_action( 'woocommerce_product_set_stock', $product );
```

### woocommerce_before_single_product

Wywoływany przed wyświetleniem strony produktu.

```php
do_action( 'woocommerce_before_single_product' );
```

**Zastosowanie:** Dodanie dodatkowych informacji przed produktem.

### woocommerce_after_single_product

Wywoływany po wyświetleniu strony produktu.

```php
do_action( 'woocommerce_after_single_product' );
```

### woocommerce_product_title

Filtr na tytuł produktu.

```php
apply_filters( 'woocommerce_product_title', $title, $product );
```

**Parametry:**
- `$title` - oryginalny tytuł
- `$product` - obiekt produktu

### woocommerce_product_get_price

Filtr na cenę produktu.

```php
apply_filters( 'woocommerce_product_get_price', $price, $product );
```

### woocommerce_short_description

Filtr na krótki opis produktu.

```php
apply_filters( 'woocommerce_short_description', $short_description );
```

### woocommerce_product_query

Filtr na zapytanie produktów.

```php
apply_filters( 'woocommerce_product_query', $query, $query_vars );
```

### woocommerce_product_tabs

Filtr na karty produktu.

```php
apply_filters( 'woocommerce_product_tabs', $tabs, $product );
```

---

## Haki Koszyka

### woocommerce_cart_contents_changed

Wywoływany, gdy zmienia się zawartość koszyka.

```php
do_action( 'woocommerce_cart_contents_changed' );
```

### woocommerce_before_cart

Wywoływany przed wyświetleniem strony koszyka.

```php
do_action( 'woocommerce_before_cart' );
```

### woocommerce_after_cart

Wywoływany po wyświetleniu strony koszyka.

```php
do_action( 'woocommerce_after_cart' );
```

### woocommerce_cart_item_removed

Wywoływany, gdy przedmiot jest usuwany z koszyka.

```php
do_action( 'woocommerce_cart_item_removed', $cart_item_key, $cart );
```

### woocommerce_add_to_cart_validation

Filtr sprawdzający, czy przedmiot może być dodany do koszyka.

```php
apply_filters( 'woocommerce_add_to_cart_validation', $passed, $product_id, $quantity );
```

### woocommerce_cart_calculate_fees

Wywoływany do dodania dodatkowych opłat do koszyka.

```php
do_action( 'woocommerce_cart_calculate_fees', $cart );
```

### woocommerce_cart_item_price

Filtr na cenę towaru w koszyku.

```php
apply_filters( 'woocommerce_cart_item_price', $price, $cart_item, $cart_item_key );
```

### woocommerce_cart_item_subtotal

Filtr na subtotal towaru w koszyku.

```php
apply_filters( 'woocommerce_cart_item_subtotal', $subtotal, $cart_item, $cart_item_key, $cart );
```

---

## Haki Zamówienia

### woocommerce_new_order

Wywoływany, gdy tworzone jest nowe zamówienie.

```php
do_action( 'woocommerce_new_order', $order_id, $order );
```

### woocommerce_order_status_changed

Wywoływany, gdy zmienia się status zamówienia.

```php
do_action( 'woocommerce_order_status_changed', $order_id, $old_status, $new_status, $order );
```

### woocommerce_process_shop_order_meta

Wywoływany podczas przetwarzania danych zamówienia w panelu administracyjnym.

```php
do_action( 'woocommerce_process_shop_order_meta', $post_id, $post );
```

### woocommerce_order_item_meta

Filtr na metadane towaru w zamówieniu.

```php
apply_filters( 'woocommerce_order_item_meta', $meta, $item );
```

### woocommerce_order_details_after_order_table

Wywoływany po wyświetleniu tabeli zamówienia.

```php
do_action( 'woocommerce_order_details_after_order_table', $order );
```

### woocommerce_email_order_items_table

Filtr na tabelę towarów w mailu zamówienia.

```php
apply_filters( 'woocommerce_email_order_items_table', $items, $order );
```

### woocommerce_thankyou

Wywoływany na stronie dziękowania po zakupie.

```php
do_action( 'woocommerce_thankyou', $order_id );
```

---

## Haki Użytkownika

### woocommerce_customer_save_address

Wywoływany, gdy użytkownik zapisuje adres.

```php
do_action( 'woocommerce_customer_save_address', $address_type );
```

### woocommerce_register_post

Wywoływany podczas rejestracji użytkownika.

```php
do_action( 'woocommerce_register_post', $username, $email, $validation_errors );
```

### woocommerce_created_customer

Wywoływany, gdy nowy użytkownik jest tworzony.

```php
do_action( 'woocommerce_created_customer', $customer_id, $new_customer_data, $password_generated );
```

### woocommerce_customer_object_updated_props

Wywoływany, gdy właściwości klienta są aktualizowane.

```php
do_action( 'woocommerce_customer_object_updated_props', $customer, $changed_props );
```

### woocommerce_account_navigation

Wywoływany w nawigacji konta użytkownika.

```php
do_action( 'woocommerce_account_navigation' );
```

---

## Haki Płatności

### woocommerce_payment_complete

Wywoływany, gdy płatność jest ukończona.

```php
do_action( 'woocommerce_payment_complete', $order_id );
```

### woocommerce_payment_complete_order_status

Filtr na status zamówienia po ukończonej płatności.

```php
apply_filters( 'woocommerce_payment_complete_order_status', $new_order_status, $order_id, $order );
```

### woocommerce_gateway_icon

Filtr na ikonę bramy płatności.

```php
apply_filters( 'woocommerce_gateway_icon', $icon, $id );
```

### woocommerce_available_payment_gateways

Filtr na dostępne bramy płatności.

```php
apply_filters( 'woocommerce_available_payment_gateways', $gateways );
```

### woocommerce_valid_order_statuses_for_payment

Filtr na statusy zamówień akceptowane dla płatności.

```php
apply_filters( 'woocommerce_valid_order_statuses_for_payment', $statuses );
```

---

## Haki Szablonów

### woocommerce_before_main_content

Wywoływany przed główną zawartością sklepu.

```php
do_action( 'woocommerce_before_main_content' );
```

### woocommerce_after_main_content

Wywoływany po głównej zawartości sklepu.

```php
do_action( 'woocommerce_after_main_content' );
```

### woocommerce_sidebar

Wywoływany w miejscu paska bocznego.

```php
do_action( 'woocommerce_sidebar' );
```

### woocommerce_archive_description

Wywoływany dla opisu archiwum produktów.

```php
do_action( 'woocommerce_archive_description' );
```

### woocommerce_results_count

Filtr na liczbę wyników.

```php
apply_filters( 'woocommerce_results_count', $count );
```

### woocommerce_catalog_orderby

Filtr na opcje sortowania katalogów.

```php
apply_filters( 'woocommerce_catalog_orderby', $orderby );
```

---

## Przykłady Kodu

### Przykład 1: Dodanie niestandardowego pola do produktu

```php
// Akcja: wyświetlanie niestandardowego pola
add_action( 'woocommerce_after_product_meta', 'my_custom_product_field' );

function my_custom_product_field() {
    $product = wc_get_product();
    $custom_field = get_post_meta( $product->get_id(), '_my_custom_field', true );
    
    if ( ! empty( $custom_field ) ) {
        echo '<div class="custom-field">';
        echo esc_html( 'Niestandardowe pole: ' . $custom_field );
        echo '</div>';
    }
}
```

### Przykład 2: Modyfikacja ceny produktu

```php
// Filtr: zmiana ceny produktu
add_filter( 'woocommerce_product_get_price', 'my_custom_price', 10, 2 );

function my_custom_price( $price, $product ) {
    // Zwiększenie ceny o 10% dla określonej kategorii
    if ( has_term( 'premium', 'product_cat', $product->get_id() ) ) {
        $price = $price * 1.10;
    }
    
    return $price;
}
```

### Przykład 3: Walidacja dodawania do koszyka

```php
// Filtr: walidacja przed dodaniem do koszyka
add_filter( 'woocommerce_add_to_cart_validation', 'my_cart_validation', 10, 3 );

function my_cart_validation( $passed, $product_id, $quantity ) {
    $product = wc_get_product( $product_id );
    
    // Sprawdzenie minimalnej ilości
    if ( $quantity < 5 ) {
        wc_add_notice( 'Minimalna ilość to 5 sztuk', 'error' );
        $passed = false;
    }
    
    return $passed;
}
```

### Przykład 4: Dodatkowa opłata do koszyka

```php
// Akcja: dodanie opłaty do koszyka
add_action( 'woocommerce_cart_calculate_fees', 'my_custom_cart_fee' );

function my_custom_cart_fee() {
    if ( is_admin() && ! defined( 'DOING_AJAX' ) ) {
        return;
    }
    
    if ( WC()->cart->get_subtotal() > 500 ) {
        WC()->cart->add_fee( 'Opłata dostawy express', 50 );
    }
}
```

### Przykład 5: Wykonanie akcji po zmianie statusu zamówienia

```php
// Akcja: niestandardowa akcja po zmianie statusu
add_action( 'woocommerce_order_status_changed', 'my_order_status_changed', 10, 4 );

function my_order_status_changed( $order_id, $old_status, $new_status, $order ) {
    if ( 'processing' === $new_status ) {
        // Wysłanie niestandardowego powiadomienia
        wp_mail(
            'admin@example.com',
            'Nowe zamówienie',
            'Zamówienie #' . $order_id . ' jest teraz w trakcie przetwarzania.'
        );
    }
}
```

### Przykład 6: Modyfikacja wyświetlania zamówienia w panelu

```php
// Akcja: dodanie informacji do zamówienia w panelu
add_action( 'woocommerce_admin_order_data_after_billing_address', 'my_custom_order_data' );

function my_custom_order_data( $order ) {
    $custom_data = get_post_meta( $order->get_id(), '_my_custom_meta', true );
    
    echo '<p><strong>Moje dane:</strong> ' . esc_html( $custom_data ) . '</p>';
}
```

### Przykład 7: Filtrowanie dostępnych metod płatności

```php
// Filtr: ukrywanie metod płatności na podstawie warunków
add_filter( 'woocommerce_available_payment_gateways', 'my_filter_gateways' );

function my_filter_gateways( $gateways ) {
    if ( is_admin() ) {
        return $gateways;
    }
    
    // Ukrycie konkretnej metody dla niezalogowanych użytkowników
    if ( ! is_user_logged_in() && isset( $gateways['bank_transfer'] ) ) {
        unset( $gateways['bank_transfer'] );
    }
    
    return $gateways;
}
```

### Przykład 8: Modyfikacja zawartości maila zamówienia

```php
// Filtr: zmiana tekstu w mailu zamówienia
add_filter( 'woocommerce_email_order_items_table', 'my_custom_email_table', 10, 4 );

function my_custom_email_table( $items, $order, $show_download_links, $mailer ) {
    // Dodanie niestandardowego tekstu
    $items .= '<p>Dziękujemy za Twój zakup!</p>';
    
    return $items;
}
```

---

## Najlepsze Praktyki

### 1. Używaj Prefiksów

Zawsze poprzedzaj nazwy funkcji i akcji unikatowym prefiksem, aby uniknąć konfliktów.

```php
// Dobrze
add_action( 'woocommerce_after_single_product', 'my_plugin_custom_action' );

// Źle
add_action( 'woocommerce_after_single_product', 'custom_action' );
```

### 2. Sprawdzaj Priorytet

Pamiętaj o priorytecie, aby upewnić się, że twoje działania są wykonywane w odpowiedniej kolejności.

```php
// Wykonane wcześniej (priorytet 5)
add_action( 'woocommerce_after_single_product', 'my_early_action', 5 );

// Wykonane jako standard (priorytet 10 - domyślny)
add_action( 'woocommerce_after_single_product', 'my_standard_action', 10 );

// Wykonane później (priorytet 15)
add_action( 'woocommerce_after_single_product', 'my_late_action', 15 );
```

### 3. Bezpieczeństwo

Zawsze weryfikuj dane wejściowe i używaj funkcji sanitacji.

```php
// Weryfikacja danych
if ( ! isset( $_POST['nonce'] ) || ! wp_verify_nonce( $_POST['nonce'], 'my_action' ) ) {
    wp_die( 'Weryfikacja nieudana' );
}

// Sanitacja danych
$safe_data = sanitize_text_field( $_POST['data'] );
```

### 4. Warunki

Zawsze sprawdzaj warunki przed wykonaniem akcji, aby zmniejszyć obciążenie.

```php
add_action( 'woocommerce_thankyou', 'my_thankyou_action' );

function my_thankyou_action( $order_id ) {
    // Sprawdzenie, czy jest to rzeczywiście strona dziękowania
    if ( ! is_order_received_page() ) {
        return;
    }
    
    // Twoja logika
}
```

### 5. Dokumentacja

Zawsze dokumentuj swoje haki i funkcje.

```php
/**
 * Dodaje niestandardowe pole do strony produktu.
 *
 * @hook woocommerce_after_single_product
 * @param void
 * @return void
 */
function my_plugin_custom_product_field() {
    // Twoja logika
}
```

### 6. Testowanie

Testuj swoje haki na wielu produktach i wariantach, aby upewnić się, że działają prawidłowo.

```php
// Test: dodaj do funkcji .wp-env
add_action( 'wp_footer', function() {
    if ( current_user_can( 'manage_options' ) && isset( $_GET['debug_hooks'] ) ) {
        error_log( 'Mój hak został wykonany' );
    }
} );
```

### 7. Wydajność

Optymalizuj swoje funkcje háków, aby nie spowalniały strony.

```php
add_action( 'woocommerce_product_query', 'my_product_filter', 10, 2 );

function my_product_filter( $query, $query_vars ) {
    // Unikaj zbędnych zapytań do bazy danych
    // Korzystaj z cache'u, jeśli to możliwe
    
    if ( false === ( $result = get_transient( 'my_cache_key' ) ) ) {
        $result = expensive_operation();
        set_transient( 'my_cache_key', $result, 12 * HOUR_IN_SECONDS );
    }
    
    return $result;
}
```

### 8. Zgodność Wersji

Zawsze sprawdzaj dokumentację WooCommerce dla swojej wersji.

```php
// Sprawdzenie wersji WooCommerce
if ( defined( 'WC_VERSION' ) && version_compare( WC_VERSION, '5.0', '>=' ) ) {
    // Nowe API
} else {
    // Stare API
}
```

---

## Zasoby i Linki

- [Oficjalna dokumentacja WooCommerce Hooks](https://woocommerce.com/document/hooks/)
- [WooCommerce Developer Reference](https://docs.woocommerce.com/)
- [WordPress Hooks Reference](https://developer.wordpress.org/plugins/hooks/)
- [WooCommerce GitHub](https://github.com/woocommerce/woocommerce)

---

**Ostatnia aktualizacja:** 27 grudnia 2025

*Dokumentacja stworzona dla WooCommerce Hooks Instrukcja*
