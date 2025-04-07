# wc-category-search-thumbnail-
<?php
/**
 * Plugin Name: WooCommerce Category Search + Thumbnail Enhancer
 * Description: Adds search boxes and category thumbnails to Product Editor and Quick Edit for WooCommerce.
 * Version: 1.1
 * Author: ChatGPT
 */

// Add search + thumbnail to Product Editor
add_action('admin_print_footer_scripts', 'wc_cat_search_thumbnail_product_editor');
function wc_cat_search_thumbnail_product_editor() {
    global $pagenow;
    if ($pagenow !== 'post.php' && $pagenow !== 'post-new.php') return;
    ?>
    <style>
        .category-search { margin-bottom: 10px; width: 100%; padding: 6px; }
        .clear-category-search { margin-bottom: 10px; }
        .categorychecklist label {
            display: flex;
            align-items: center;
            gap: 6px;
        }
        .categorychecklist img.cat-thumb {
            width: 20px;
            height: 20px;
            object-fit: cover;
            border-radius: 3px;
        }
    </style>
    <script>
        jQuery(document).ready(function($) {
            var categoryBox = $('#product_catdiv .inside');
            if (!categoryBox.length) return;

            categoryBox.prepend('<input type="text" class="category-search" placeholder="ক্যাটেগরি সার্চ করুন..."/>            <button type="button" class="button clear-category-search">Clear Search</button>');

            var debounceTimeout;
            $('.category-search').on('keyup', function() {
                clearTimeout(debounceTimeout);
                debounceTimeout = setTimeout(() => {
                    var searchTerm = $(this).val().toLowerCase();
                    categoryBox.find('.categorychecklist li').each(function() {
                        var text = $(this).text().toLowerCase();
                        var checked = $(this).find('input[type="checkbox"]').is(':checked');
                        $(this).toggle(text.includes(searchTerm) || checked);
                    });
                }, 300);
            });

            $('.clear-category-search').on('click', function() {
                $('.category-search').val('');
                categoryBox.find('.categorychecklist li').show();
            });

            // Add thumbnails
            categoryBox.find('.categorychecklist li').each(function() {
                var li = $(this);
                var input = li.find('input[type="checkbox"]');
                var catID = input.val();

                $.ajax({
                    url: ajaxurl,
                    data: { action: 'get_wc_category_thumbnail', term_id: catID },
                    type: 'POST',
                    success: function(res) {
                        if (res && res.url) {
                            li.find('label').prepend('<img class="cat-thumb" src="' + res.url + '" alt="cat-thumb"/>');
                        }
                    }
                });
            });
        });
    </script>
    <?php
}

// Add search + thumbnails to Quick Edit
add_action('admin_footer', 'wc_cat_search_thumbnail_quick_edit');
function wc_cat_search_thumbnail_quick_edit() {
    $screen = get_current_screen();
    if ($screen->post_type !== 'product') return;
    ?>
    <script>
    jQuery(document).on('click', '.editinline', function() {
        setTimeout(function() {
            const quickEditBox = jQuery('.inline-edit-categories');
            if (quickEditBox.find('#quick-edit-cat-search').length === 0) {
                quickEditBox.prepend('<input type="text" id="quick-edit-cat-search" placeholder="ক্যাটেগরি সার্চ করুন..." style="margin-bottom:10px; width: 100%;">');
            }

            jQuery('#quick-edit-cat-search').on('keyup', function() {
                const filter = jQuery(this).val().toLowerCase();
                quickEditBox.find('label').each(function() {
                    const text = jQuery(this).text().toLowerCase();
                    jQuery(this).toggle(text.includes(filter));
                });
            });
        }, 200);
    });
    </script>
    <?php
}

// Handle AJAX thumbnail fetch
add_action('wp_ajax_get_wc_category_thumbnail', function() {
    $term_id = absint($_POST['term_id']);
    $thumb_id = get_term_meta($term_id, 'thumbnail_id', true);
    $image_url = wp_get_attachment_url($thumb_id);
    wp_send_json(['url' => $image_url]);
});

