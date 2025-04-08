<?php
/**
 * Plugin Name: WP Autoload Cleaner
 * Description: A tool to identify and manage autoloaded options in the wp_options table.
 * Version: 1.0
 * Author: Dylan Jackson
 */

if ( ! defined( 'ABSPATH' ) ) exit; // Exit if accessed directly.

// Add the plugin to the admin menu
add_action( 'admin_menu', 'my_autoload_cleaner_menu' );
function my_autoload_cleaner_menu() {
    add_menu_page( 
        'Autoload Cleaner', 
        'Autoload Cleaner', 
        'manage_options', 
        'autoload-cleaner', 
        'my_autoload_cleaner_page' 
    );
}

// Render the admin page
function my_autoload_cleaner_page() {
      // Get the current tab from the URL or default to 'all'
      $current_tab = isset( $_GET['tab'] ) ? sanitize_text_field( $_GET['tab'] ) : 'all';

      // Render the tabs
      echo '<div class="wrap">';
      echo '<h1>WP Autoload Cleaner</h1>';
      echo '<h2 class="nav-tab-wrapper">';
      echo '<a href="' . admin_url( 'admin.php?page=autoload-cleaner&tab=all' ) . '" class="nav-tab ' . ( $current_tab === 'all' ? 'nav-tab-active' : '' ) . '">All Autoloaded Options</a>';
      echo '<a href="' . admin_url( 'admin.php?page=autoload-cleaner&tab=unused' ) . '" class="nav-tab ' . ( $current_tab === 'unused' ? 'nav-tab-active' : '' ) . '">Potentially Unused Options</a>';
      echo '<a href="' . admin_url( 'admin.php?page=autoload-cleaner&tab=transients' ) . '" class="nav-tab ' . ( $current_tab === 'transients' ? 'nav-tab-active' : '' ) . '">Transients</a>';
      echo '</h2>';
  
      // Display content for the selected tab
      if ( $current_tab === 'all' ) {
          echo '<h2>All Autoloaded Options</h2>';
          my_autoload_cleaner_scan();
      } elseif ( $current_tab === 'unused' ) {
          echo '<h2>Potentially Unused Options</h2>';
          display_unused_autoload_options();
      } elseif ( $current_tab === 'transients' ) {
          echo '<h2>Transients</h2>';
          display_transients();
      }
  
      echo '</div>';
}

// Scan and display all autoloaded options
function my_autoload_cleaner_scan() {
  global $wpdb;

  echo '<div style="border-left: 4px solid #ffba00; background: #fffbe6; padding: 10px; margin-bottom: 20px;">';
  echo '<p><strong>Warning:</strong> Deleting autoloaded options can affect your site’s functionality. Please proceed with caution and only remove options you are certain are unused or no longer needed.</p>';
  echo '</div>';

  // Fetch and display all autoloaded options as before
  $results = $wpdb->get_results( "
      SELECT option_name, LENGTH(option_value) AS size
      FROM {$wpdb->options}
      WHERE autoload = 'yes'
      ORDER BY size DESC
  " );

  echo '<table class="widefat fixed">';
  echo '<thead><tr><th>Option Name</th><th>Size (Bytes)</th><th>Actions</th></tr></thead>';
  echo '<tbody>';

  foreach ( $results as $row ) {
      echo '<tr>';
      echo '<td>' . esc_html( $row->option_name ) . '</td>';
      echo '<td>' . esc_html( $row->size ) . '</td>';
      echo '<td><a href="?delete_option=' . urlencode( $row->option_name ) . '" class="button">Delete</a></td>';
      echo '</tr>';
  }

  echo '</tbody></table>';
}


// Handle deletion of autoloaded options
if ( isset( $_GET['delete_option'] ) ) {
    add_action( 'admin_init', 'my_autoload_cleaner_delete' );
}

function my_autoload_cleaner_delete() {
    global $wpdb;

    $option_name = sanitize_text_field( $_GET['delete_option'] );

    // Verify the option exists before attempting to delete
    if ( get_option( $option_name ) !== false ) {
        $wpdb->query( $wpdb->prepare( "DELETE FROM {$wpdb->options} WHERE option_name = %s", $option_name ) );
    }

    wp_redirect( admin_url( 'admin.php?page=autoload-cleaner' ) );
    exit;
}

// Detect potentially unused autoloaded options
function detect_unused_autoload_options() {
  global $wpdb;

    $autoloaded_options = $wpdb->get_results( "
        SELECT option_name, LENGTH(option_value) AS size
        FROM {$wpdb->options}
        WHERE autoload = 'yes'
    " );

    $active_plugins = get_option( 'active_plugins', [] );
    $active_theme = wp_get_theme();
    $plugin_prefixes = array_map( function( $plugin ) {
        return explode( '/', $plugin )[0];
    }, $active_plugins );
    $theme_prefix = $active_theme->get_stylesheet();

    // Expanded list of critical core options
    $core_options = array(
      'siteurl',
      'home',
      'blogname',
      'blogdescription',
      'users_can_register',
      'admin_email',
      'start_of_week',
      'use_balanceTags',
      'use_smilies',
      'require_name_email',
      'comments_notify',
      'posts_per_rss',
      'rss_use_excerpt',
      'mailserver_url',
      'mailserver_login',
      'mailserver_pass',
      'mailserver_port',
      'default_category',
      'default_comment_status',
      'default_ping_status',
      'default_pingback_flag',
      'posts_per_page',
      'date_format',
      'time_format',
      'links_updated_date_format',
      'comment_moderation',
      'moderation_notify',
      'permalink_structure',
      'rewrite_rules',
      'hack_file',
      'blog_charset',
      'moderation_keys',
      'active_plugins',
      'category_base',
      'ping_sites',
      'comment_max_links',
      'gmt_offset',
      'default_email_category',
      'recently_edited',
      'template',
      'stylesheet',
      'comment_registration',
      'html_type',
      'use_trackback',
      'default_role',
      'db_version',
      'uploads_use_yearmonth_folders',
      'upload_path',
      'blog_public',
      'default_link_category',
      'show_on_front',
      'tag_base',
      'show_avatars',
      'avatar_rating',
      'upload_url_path',
      'thumbnail_size_w',
      'thumbnail_size_h',
      'thumbnail_crop',
      'medium_size_w',
      'medium_size_h',
      'avatar_default',
      'large_size_w',
      'large_size_h',
      'image_default_link_type',
      'image_default_size',
      'image_default_align',
      'close_comments_for_old_posts',
      'close_comments_days_old',
      'thread_comments',
      'thread_comments_depth',
      'page_comments',
      'comments_per_page',
      'default_comments_page',
      'comment_order',
      'sticky_posts',
      'widget_categories',
      'widget_text',
      'widget_rss',
      'uninstall_plugins',
      'timezone_string',
      'page_for_posts',
      'page_on_front',
      'default_post_format',
      'link_manager_enabled',
      'finished_splitting_shared_terms',
      'site_icon',
      'medium_large_size_w',
      'medium_large_size_h',
      'wp_page_for_privacy_policy',
      'show_comments_cookies_opt_in',
      'admin_email_lifespan',
      'disallowed_keys',
      'comment_previously_approved',
      'auto_plugin_theme_update_emails',
      'auto_update_core_dev',
      'auto_update_core_minor',
      'auto_update_core_major',
      'wp_force_deactivated_plugins',
      'wp_attachment_pages_enabled',
      'initial_db_version',
      'can_compress_scripts',
      'db_upgraded',
  );
  

    $unused_options = [];
    foreach ( $autoloaded_options as $option ) {
        $is_used = false;

        // Check if the option matches active plugins or theme
        foreach ( $plugin_prefixes as $prefix ) {
            if ( strpos( $option->option_name, $prefix ) === 0 ) {
                $is_used = true;
                break;
            }
        }

        if ( strpos( $option->option_name, $theme_prefix ) === 0 ) {
            $is_used = true;
        }

        // Exclude critical core options
        if ( in_array( $option->option_name, $core_options ) ) {
            $is_used = true;
        }

        // Exclude options marked as safe
        if ( is_option_safe( $option->option_name ) ) {
            continue;
        }

        if ( ! $is_used ) {
            $unused_options[] = $option;
        }
    }

    return $unused_options;
}

// Display potentially unused autoloaded options
function display_unused_autoload_options() {
  echo '<div style="border-left: 4px solid #ffba00; background: #fffbe6; padding: 10px; margin-bottom: 20px;">';
  echo '<p><strong>Warning:</strong> Removing potentially unused options could break functionality. Only delete options if you are confident they are no longer needed.</p>';
  echo '</div>';

  $unused_options = detect_unused_autoload_options();

  if ( empty( $unused_options ) ) {
      echo '<p>All autoloaded options appear to be in use.</p>';
      return;
  }

  echo '<table class="widefat fixed">';
  echo '<thead><tr><th>Option Name</th><th>Size (Bytes)</th><th>Status</th><th>Action</th></tr></thead>';
  echo '<tbody>';

  foreach ( $unused_options as $option ) {
      $safe_label = is_option_safe( $option->option_name ) ? 'Safe' : 'Unused';

      echo '<tr>';
      echo '<td>' . esc_html( $option->option_name ) . '</td>';
      echo '<td>' . esc_html( $option->size ) . '</td>';
      echo '<td>' . esc_html( $safe_label ) . '</td>';
      echo '<td>';
      if ( is_option_safe( $option->option_name ) ) {
          echo '<span class="button disabled">Marked Safe</span>';
      } else {
          echo '<a href="?mark_safe=' . urlencode( $option->option_name ) . '" class="button">Mark as Safe</a>';
          echo '<a href="?delete_option=' . urlencode( $option->option_name ) . '" class="button">Delete</a>';
      }
      echo '</td>';
      echo '</tr>';
  }

  echo '</tbody></table>';
}



// Handle marking options as safe
if ( isset( $_GET['mark_safe'] ) ) {
    add_action( 'admin_init', function() {
        $option_name = sanitize_text_field( $_GET['mark_safe'] );

        // Verify the option exists before marking it safe
        if ( get_option( $option_name ) !== false ) {
            mark_option_safe( $option_name );
        } else {
            wp_die( 'Invalid option specified.' );
        }
    } );
}

function mark_option_safe( $option_name ) {
    $safe_options = get_option( 'safe_autoload_options', [] );
    $safe_options[] = $option_name;
    update_option( 'safe_autoload_options', array_unique( $safe_options ) );
    wp_redirect( admin_url( 'admin.php?page=autoload-cleaner' ) );
    exit;
}

// Check if an option is marked as safe
function is_option_safe( $option_name ) {
    $safe_options = get_option( 'safe_autoload_options', [] );
    return in_array( $option_name, $safe_options );
}

function display_transients() {
  global $wpdb;

  // Get filter from query string
  $filter = isset( $_GET['filter'] ) ? sanitize_text_field( $_GET['filter'] ) : 'all';

  // Build query based on filter
  if ( $filter === 'expired' ) {
      $transients = $wpdb->get_results( "
          SELECT option_name, option_value
          FROM {$wpdb->options}
          WHERE option_name LIKE '_transient_%'
          AND option_name NOT LIKE '_transient_timeout_%'
          AND (
              SELECT option_value
              FROM {$wpdb->options}
              WHERE option_name = CONCAT('_transient_timeout_', SUBSTRING(option_name, 12))
          ) < " . time() . "
      " );
  } else {
      // Default: fetch all transients
      $transients = $wpdb->get_results( "
          SELECT option_name, option_value
          FROM {$wpdb->options}
          WHERE option_name LIKE '_transient_%'
          AND option_name NOT LIKE '_transient_timeout_%'
      " );
  }

  // Display filter links
  echo '<div class="tablenav top">';
  echo '<ul class="subsubsub">';
  echo '<li><a href="' . admin_url( 'admin.php?page=autoload-cleaner&tab=transients&filter=all' ) . '" class="' . ( $filter === 'all' ? 'current' : '' ) . '">All</a> | </li>';
  echo '<li><a href="' . admin_url( 'admin.php?page=autoload-cleaner&tab=transients&filter=expired' ) . '" class="' . ( $filter === 'expired' ? 'current' : '' ) . '">Expired</a></li>';
  echo '</ul>';
  echo '</div>';

  if ( empty( $transients ) ) {
      echo '<p>No transients found.</p>';
      return;
  }

  // Display transients in a table
  echo '<table class="widefat fixed">';
  echo '<thead><tr><th>Transient Name</th><th>Expiration</th><th>Action</th></tr></thead>';
  echo '<tbody>';

  foreach ( $transients as $transient ) {
      // Check for expiration time
      $timeout_name = '_transient_timeout_' . substr( $transient->option_name, 11 );
      $expiration = $wpdb->get_var( $wpdb->prepare( "
          SELECT option_value
          FROM {$wpdb->options}
          WHERE option_name = %s
      ", $timeout_name ) );

      // Determine expiration status
      if ( $expiration ) {
          $expiration_time = date( 'Y-m-d H:i:s', $expiration );
          $status = ( time() > $expiration ) ? 'Expired' : 'Expires at ' . $expiration_time;
      } else {
          $status = 'No Expiration';
      }

      echo '<tr>';
      echo '<td>' . esc_html( $transient->option_name ) . '</td>';
      echo '<td>' . esc_html( $status ) . '</td>';
      echo '<td>';
      echo '<a href="?delete_transient=' . urlencode( $transient->option_name ) . '" class="button">Delete</a>';
      echo '</td>';
      echo '</tr>';
  }

  echo '</tbody></table>';

  // Add bulk action buttons
  echo '<form method="post" style="margin-top: 10px;">';
  echo '<input type="hidden" name="delete_all_transients" value="1">';
  echo '<button type="submit" class="button button-primary">Remove All Transients</button>';
  echo '</form>';

  echo '<form method="post" style="margin-top: 10px;">';
  echo '<input type="hidden" name="delete_expired_transients" value="1">';
  echo '<button type="submit" class="button">Remove Expired Transients</button>';
  echo '</form>';
}


if ( isset( $_GET['delete_transient'] ) ) {
  add_action( 'admin_init', function() {
      $transient_name = sanitize_text_field( $_GET['delete_transient'] );

      // Delete transient
      if ( strpos( $transient_name, '_transient_' ) === 0 ) {
          delete_option( $transient_name );
      }

      // Redirect back to the transients tab
      wp_redirect( admin_url( 'admin.php?page=autoload-cleaner&tab=transients' ) );
      exit;
  } );
}

if ( isset( $_POST['delete_expired_transients'] ) ) {
  add_action( 'admin_init', function() {
      global $wpdb;

      // Bulk delete only expired transients
      $wpdb->query( "
          DELETE FROM {$wpdb->options}
          WHERE option_name LIKE '_transient_%'
          AND option_name NOT LIKE '_transient_timeout_%'
          AND (
              SELECT option_value
              FROM {$wpdb->options}
              WHERE option_name = CONCAT('_transient_timeout_', SUBSTRING(option_name, 12))
          ) < " . time() . "
      " );

      wp_redirect( admin_url( 'admin.php?page=autoload-cleaner&tab=transients&filter=expired' ) );
      exit;
  } );
}

if ( isset( $_POST['delete_all_transients'] ) ) {
  add_action( 'admin_init', function() {
      global $wpdb;

      // Bulk delete all transients
      $wpdb->query( "
          DELETE FROM {$wpdb->options}
          WHERE option_name LIKE '_transient_%'
      " );

      wp_redirect( admin_url( 'admin.php?page=autoload-cleaner&tab=transients' ) );
      exit;
  } );
}

add_action( 'admin_footer', function() {
  echo '<script>
      document.addEventListener("DOMContentLoaded", function() {
          const deleteButtons = document.querySelectorAll("a.button[href*=\'delete_option\']");
          deleteButtons.forEach(button => {
              button.addEventListener("click", function(event) {
                  const confirmed = confirm("Are you sure you want to delete this option? This action cannot be undone.");
                  if (!confirmed) {
                      event.preventDefault();
                  }
              });
          });
      });
  </script>';
} );
