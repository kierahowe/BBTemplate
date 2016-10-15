<?php
/**
 * @package BBTemplate
 * @version 1.0
 */
/*
Plugin Name: bbtemplate
Plugin URI: https://github.com/kierahowe/BBTemplate
Description: BBPress Templater - quick creation of BBPress Forums from a predefined template.
Author: Kiera Howe
Version: 1.0
Author URI: http://www.kierahowe.com
*/

add_action( 'wp_ajax_bbtcreatenew', 'bbt_createnew_callback' );

function bbt_cfor1 ($parent, $name, $content = "") { 
	$forum_data = array(
		'post_status'    => bbp_get_public_status_id(),
		'post_type'      => bbp_get_forum_post_type(),
		'post_author'    => bbp_get_current_user_id(),
		'post_password'  => '',
		'post_content'   => $content,
		'post_title'     => $name,
		'menu_order'     => 0,
		'comment_status' => 'closed'
	);

	if ($parent != "") { 
		$forum_data['post_parent'] = $parent;
	}

	return bbp_insert_forum( $forum_data );
}

function bbt_reqCreate ($tree, $parent, $lev = 0) { 
	if ($lev > 8) { return; }

	foreach ($tree as $n => $v) { 
		$pnum = bbt_cfor1 ($parent, $v['name']);
		$t = "";
		for ($i = 0; $i < $lev; $i ++) { $t .= "   "; }
		print "Create: " .$t . $v['name'] . "\n";
		if (count($v['children']) > 0) { 
			bbt_reqCreate ($v['children'], $pnum, $lev + 1 );
		}
	}
}

function bbt_createnew_callback() {
	$name = $_POST['name'];
	$id = $_POST['id'];
	if (!function_exists('bbp_insert_forum')) { 
		print "You must install BBPress for this to work";
		wp_die();
	}
	$forumtree = json_decode(get_post_meta($id, 'forumtree', true), true);
	if ($forumtree !== null) { 
		$par = bbt_cfor1 ("", $name);
		print "Create " . $name . "\n";
		bbt_reqCreate ($forumtree, $par, 0);
	} else { 
		print "Invalid tree input";
	}


	wp_die(); 
}

function bbt_plugin_admin_init() {
    // Register our script.
    wp_enqueue_style( 'style-name',  plugins_url( '/jqtree.css', __FILE__ ) );
	wp_enqueue_script( 'script-name', plugins_url( '/tree.jquery.js', __FILE__ ), array( 'jquery' ) );
//    wp_register_script( 'my-plugin-script', plugins_url( '/tree.jquery.js', __FILE__ ) );
}
add_action( 'admin_init', 'bbt_plugin_admin_init' );

function bbt_edit_save_postdata($post_id) {
	# Is the current user is authorised to do this action?
    if (
    	($_POST['post_type'] === 'bbtemplates') && 
    	(current_user_can('edit_page', $post_id) || current_user_can('edit_post', $post_id))
    	) { // If it's a page, OR, if it's a post, can the user edit it? 
    	//print_r ($_POST);
    	$forums = $_POST['forumtree'];
		add_post_meta($post_id, 'forumtree', $forums, true) OR update_post_meta($post_id, 'forumtree', $forums);

	}
}

function bbt_outtestitems ($post) { 
	$forumtree = get_post_meta($post->ID, 'forumtree', true);
	if (json_decode($forumtree, true) === null) { 
		$forumtree = '[]';
	}
?>
<style>
#tree1 .edit {
	margin-left: 20px;
}
#tree1 .delete {
	margin-left: 5px;
}
#tree1 a { 
	text-decoration: none;
	color: grey;
}
#tree1 li { 
}
</style>
<input type=hidden name="forumtree" id="forumtree" value="<?php echo $forumtree; ?>">
<div id="tree1"></div>
<input type=text id="newname">
<input type=button onClick="addNewItem();" value="Add">
<script>
var data = <?php echo $forumtree; ?>;
var jq; 

jQuery( document ).ready( function( $ ) {
	jq = $;
	$(function() {
	    $('#tree1').tree({
			data: data,
			autoOpen: true,
			dragAndDrop: true,
			onCreateLi: function(node, $li) {
				// Append a link to the jqtree-element div.
				// The link has an url '#node-[id]' and a data property 'node-id'.
				$li.find('.jqtree-element').append(
					'<a href="#node-'+ node.id +'" class="edit" data-node-id="'+
					node.id +'"><span data-node-id="'+
					node.id +'" id="' + node.id + '" class="dashicons dashicons-edit"></span></a>' +
					'<a href="#node-'+ node.id +'" class="delete" data-node-id="'+
					node.id +'"><span data-node-id="'+
					node.id +'" id="' + node.id + '" class="dashicons dashicons-trash"></span></a>'
				);
			}
	    });
	    $('#tree1').bind(
		    'tree.move',
		    function(event) {
				event.preventDefault();
				// do the move first, and _then_ POST back.
				event.move_info.do_move();
		        document.getElementById ('forumtree').value = $(this).tree('toJson');
		    }
		);
	});

    $('#tree1').on(
        'click', '.edit',
        function(e) {
            // Get the id from the 'node-id' data property
            var id = $(e.target).data('node-id');
        	var node = $('#tree1').tree('getNodeById', id);

            var name = prompt("Please enter the new name", node.name);
			if (name != null) {
				$('#tree1').tree('updateNode', node, { 'name': name });
			}
			document.getElementById ('forumtree').value = $(this).tree('toJson');
        }
    );
    $('#tree1').on(
        'click', '.delete',
        function(e) {
        	if (confirm ("Are you sure you want to delete that node and all its children?")) { 
        		var id = $(e.target).data('node-id');
        		var node = $('#tree1').tree('getNodeById', id);
	            $('#tree1').tree('removeNode', node);
	            document.getElementById ('forumtree').value = $(this).tree('toJson');
	        }
        }
    );
});

function reqNodes (node, lev) { 
	var max = -1;
	if (lev > 8) { 
		return 0;
	}

	for (var t in node) { 
		if (node[t].id > max) { max = node[t].id; }
		if (node[t].children.length > 0) { 
			var o = reqNodes (node[t].children, lev + 1);
			if (o > max) { max = o; }
		}
	}

	return max;
}

function addNewItem () { 

	var nodeid = reqNodes (jQuery('#tree1').tree('getTree').children) + 1;

	jq('#tree1').tree(
	    'appendNode',
	    {
	        name: document.getElementById ('newname').value,
	        id: nodeid
	    }
	);
	document.getElementById ('forumtree').value = jQuery('#tree1').tree('toJson');
}

</script>
<?php
}

function bbt_runit($post) { 
?>
	New Item: <input type=text name="fname" id="fname"> 
	<input type=button value="go" onClick="createnew()" value="Create New"/>
	<br><br>
	<div style="border: 1px solid #333333; background-color: #dddddd; padding: 10px; display: inline-block">
		A new forum with the specified name, containing all the forums in the above tree, 
		will be created in the root of your site.
	</div>	
<script>
function createnew() { 
	var name = document.getElementById ('fname').value;
	var data = {
		'action': 'bbtcreatenew',
		'name': name, 
		'id': <?php echo $post->ID; ?>
	};
	jQuery.post(ajaxurl, data, function(response) {
		alert(response);
	});
}
</script>
<?php 
}

function bbt_tester_edit ($post_type) { 
	if ($post_type == 'bbtemplates') { 
		add_meta_box(
			'bbt_testitemsbox', // HTML 'id' attribute of the edit screen section.
			'Forums',              // Title of the edit screen section, visible to user.
			'bbt_outtestitems', // Function that prints out the HTML for the edit screen section.
			null,          // The type of Write screen on which to show the edit screen section.
			'normal',          // The part of the page where the edit screen section should be shown.
			'default'               // The priority within the context where the boxes should show.
		);
		add_meta_box(
			'bbt_runit', // HTML 'id' attribute of the edit screen section.
			'Create Forums from this template',              // Title of the edit screen section, visible to user.
			'bbt_runit', // Function that prints out the HTML for the edit screen section.
			null,          // The type of Write screen on which to show the edit screen section.
			'normal',          // The part of the page where the edit screen section should be shown.
			'default'               // The priority within the context where the boxes should show.
		);
	}
}
add_action('save_post', 'bbt_edit_save_postdata');
add_action( 'add_meta_boxes', 'bbt_tester_edit');

///////////////////////////////////////
///  Add class fields to the registration and profile

// Register Custom Class
function bbt_custom_classes() {
	$labels = array(
		'name'                  => _x( 'BBTemplates', 'BBtemplate General Name', 'text_domain' ),
		'singular_name'         => _x( 'BBTemplate', 'BBtemplate Singular Name', 'text_domain' ),
		'menu_name'             => __( 'BBTemplates', 'text_domain' ),
		'name_admin_bar'        => __( 'BBTemplate', 'text_domain' ),
		'archives'              => __( 'Item Archives', 'text_domain' ),
		'parent_item_colon'     => __( 'Parent Item:', 'text_domain' ),
		'all_items'             => __( 'BBTemplates', 'text_domain' ),
		'add_new_item'          => __( 'Add New Item', 'text_domain' ),
		'add_new'               => __( 'Add New', 'text_domain' ),
		'new_item'              => __( 'New Item', 'text_domain' ),
		'edit_item'             => __( 'Edit Item', 'text_domain' ),
		'update_item'           => __( 'Update Item', 'text_domain' ),
		'view_item'             => __( 'View Item', 'text_domain' ),
		'search_items'          => __( 'Search Item', 'text_domain' ),
		'not_found'             => __( 'Not found', 'text_domain' ),
		'not_found_in_trash'    => __( 'Not found in Trash', 'text_domain' ),
		'featured_image'        => __( 'Featured Image', 'text_domain' ),
		'set_featured_image'    => __( 'Set featured image', 'text_domain' ),
		'remove_featured_image' => __( 'Remove featured image', 'text_domain' ),
		'use_featured_image'    => __( 'Use as featured image', 'text_domain' ),
		'insert_into_item'      => __( 'Insert into item', 'text_domain' ),
		'uploaded_to_this_item' => __( 'Uploaded to this item', 'text_domain' ),
		'items_list'            => __( 'Items list', 'text_domain' ),
		'items_list_navigation' => __( 'Items list navigation', 'text_domain' ),
		'filter_items_list'     => __( 'Filter items list', 'text_domain' ),
	);
	$args = array(
		'label'                 => __( 'BBtemplate', 'text_domain' ),
		'description'           => __( 'BBtemplate Description', 'text_domain' ),
		'labels'                => $labels,
		'supports'              => array('title'),
		'taxonomies'            => array( ),
		'hierarchical'          => false,
		'public'                => true,
		'show_ui'               => true,
		'show_in_menu'          => true,
		'menu_position'         => 50,
		'show_in_admin_bar'     => true,
		'show_in_nav_menus'     => true,
		'can_export'            => true,
		'has_archive'           => true,		
		'exclude_from_search'   => false,
		'publicly_queryable'    => true,
		'capability_type'       => 'page',
	);
	register_post_type( 'bbtemplates', $args );

}
add_action( 'init', 'bbt_custom_classes', 0 );
