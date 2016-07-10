# Arconix Post Type Helper

Register WordPress custom post types and taxonomies using proper object-oriented design.

## Description

Adding Custom Post Types, especially multiple types in the same project, involves a great deal of copying and pasting. Additionally, most implementations that use classes end up using them as a makeshift namespace that goes against proper object-oriented programming. The helper classes in this repo make it easier to register custom post types and their taxonomies as well as simplify the admin-side changes that are typically made with public post types in a way that is consistent with proper OOP.

## File Structure

 * `Arconix_CPT_Register` - Class for registering post types. All user-facing work will be done through the `add()` function.
 * `Arconix_Taxonomy_Register` - Class for registering taxonomies. Like the CPT class, user-facing work will be done through the `add()` function.
 * `Arconix_CPT_Admin` - Abstract class for customizing the admin-side functionality. There are two abstract methods that must be defined, in addition to multiple methods available to be overridden if needed. 


## Requirements
 * PHP 5.2+

## Installation

1. Copy the `arconix` files into your plugin folder, ideally into a subdirectory with like files (such as `includes`).
2. Make the classes available. You can `require_once` each file, if the structural element doesn't exist, or just use an autoloader (renaming anything as needed if following PSR-0/4).

## Usage

### Post Type Registration - `Arconix_CPT_Register`
The most basic implementation possible

    $cpt = new Arconix_CPT_Register();
    
    $cpt->add( 'books' );

That's it. We've registered a new `Books` post type using the default settings (setting the post type to public) and the textdomain (for translations) is set to `default`. If you won't be translating your plugin or making it available on wordpress.org, you can ignore that argument.

If we want to get a bit more advanced, we can define the singular and plural versions of the Post Type name, which is espcially helpful if the pluralized version of the post type name does not end in 's' or we want to provide a namespaced post type name.

    $cpt = new Arconix_CPT_Register();
    
    $pt_name = array (
        'post_type_name' => 'ac_people',
        'plural' => 'People',
        'singular' => 'Person'
    );
    
    $cpt->add( $pt_name );

If the post type name and plural name are the same you can safely leave the 'plural' definition out of the array and the class will use the post type name to create a human-friendly plural version for you.

Creating the settings for the post type is no more difficult. Simply create an array of arguments and those will be passed along with the post type name for registration.

    $cpt = new Arconix_CPT_Register();
    
    $settings = array(
        'menu_position' => 40,
        'menu_icon' => 'dashicons-book-alt',
        'supports' => array( 'title', 'editor', 'revisions', 'thumbnail' )
    );
    
    $cpt->add( 'books', $settings );

### Taxonomy Registration - `Arconix_Taxonomy_Register`

Using this class follows the same format as adding a custom post type. The only difference is the additional argument to the `add()` function.

    public function add( $taxonomy_names, $post_type_name = null, $settings = array(), $textdomain = 'default' );

To attach the taxonomy to an registered post type (existing or one you've created), the post type name must be passed as the second argument to the function otherwise the taxonomy will be registered but not linked.

    $tax = new Arconix_Taxonomy_Register();
    
    $tax->add( 'genre', 'books' );

### Customizing the Admin - `Arconix_CPT_Admin`
This abstract class must be extended (in the same way `WP_Widget` is extended when creating a new widget). When extending this class there are three functions that must be defined: `__construct()`, `columns_define()` and `column_value()`. These functions, respectively, initialize the class, control what columns show up on the admin screen and what content will be displayed in each column. The remaining functions in the abstract class will run as is unless there is need to customize their behavior. A basic implementation would look like the following:

    class Arconix_Books_Admin extends Arconix_CPT_Admin {
        // The only thing this function does in this example is pass the arguments to the abstract parent class for execution
        public function __construct( $post_type_name, $textdomain = 'default' ) {
            parent::__construct( $post_type_name, $textdomain );
        }
        
        // This function would define which columns are available on the Admin screen
        public function columns_define( $columns ) {
            ...
        }
        
        // This function provides the value for each column
        public function column_value( $column ) {
            ...
        }
    }

The extended class can add new functionality that isn't present in the abstract class:

    class Arconix_Books_Admin extends Arconix_CPT_Admin {
        ...
        
        // This function customizes the placeholder text on the Add New screen
        public function title_text( $title ) {
            $screen = get_current_screen();

	 	    if ( $this->post_type_name == $screen->post_type )
			    return __( 'Enter my Book Title here...', $this->textdomain );
        }
    }

That function must be hooked in to WordPress to have any effect, so we define an `init()` function in the extended class to override the abstract class behavior:

    class Arconix_Books_Admin extends Arconix_CPT_Admin {
        ...
        
        public function init() {
            add_filter( 'enter_title_here',	array( $this, 'title_text' ) );
        
            // Calling the abstract parent class `init()` function ensures the default functionality is left 
            // in place and doesn't need to be redefined here
            parent::init();
        }
    }

Once all the customization work is done, the class needs to be instantiated. This is achieved with one instruction:
    
    $acb = new Arconix_Books_Admin( 'books' );

As we saw earlier, the class accepts two arguments, the post type we're customizing and the texdomain (optional) for translations.

## Contribute

Issues and Pull Requests are welcomed. Please submit against the 'develop' branch. Code should be documented and follow the WordPress coding standards

## License

GPL-2.0+

## Credits
Built by [John Gardner](http://twitter.com/j_gardner)
