# Drupal 8 Snippets
Useful snippets

## GET TERM OR NODE OBJECT CURRENT PAGE
```
// TERM:
    $term = \Drupal::routeMatch()->getParameter('taxonomy_term');
    if ($term instanceof TermInterface && $term->getVocabularyId() == 'catalog') {
      if ($term->hasField('field_level')) {
        if ($term->get('field_level')->value != 1) {
          $level = TRUE;
        }
      }
    }

//NODE:
  $node = \Drupal::routeMatch()->getParameter('node');
  if ($node instanceof NodeInterface) {
    if ($node->hasField('field_hero_image')) {
      if (!$node->get('field_hero_image')->isEmpty()) {
        $hero_image_exists = TRUE;
      }
    }
  }
```


## How to get the current language

```
// To get the lanuage code:
$language = \Drupal::languageManager()->getCurrentLanguage()->getId();

//To get the language name:
$language =  \Drupal::languageManager()->getCurrentLanguage()->getName();
``` 

## Load block
```
// Load block and render
$socialNetworksBlock = \Drupal\block_content\Entity\BlockContent::load(7);
$variables['social_networks'] = \Drupal::entityTypeManager()
          ->getViewBuilder('block_content')
          ->view($socialNetworksBlock);
``` 

## Print block programmatically

```
$block = \Drupal\block_content\Entity\BlockContent::load('BLOCK_ID');
$block_array = \Drupal::entityTypeManager()->getViewBuilder('block_content')->view($block);
``` 

## Add a message to Drupal 8 Log system (good bye watchdog :))

```
// Redirect to the front page:
return new \Symfony\Component\HttpFoundation\RedirectResponse(\Drupal::url('<front>'));

// Redirect to a route path (user page)
return new \Symfony\Component\HttpFoundation\RedirectResponse(\Drupal::url('user.page'));

// To a internal path
 return new \Symfony\Component\HttpFoundation\RedirectResponse('/node/17/edit');
// OR
   return new \Symfony\Component\HttpFoundation\RedirectResponse(\Drupal\Core\Url::fromUserInput('/node/17/edit')->toString());
``` 

## Page Redirection

```
\Drupal::logger("mymodule")->alert("Message");
\Drupal::logger("mymodule")->critical("Message");
\Drupal::logger("mymodule")->debug("Message");
\Drupal::logger("mymodule")->emergency("Message");
\Drupal::logger("mymodule")->error("Message");
\Drupal::logger("mymodule")->info("Message");
\Drupal::logger("mymodule")->notice("Message");
\Drupal::logger("mymodule")->warning("Message");
``` 

## Debug with logger

```
\Drupal::logger('some_channel_name')->warning('<pre><code>' . print_r($responseObj, TRUE) . '</code></pre>');
``` 

## Redirect users to some page when submitting a form

```
// Redirect to an internal Drupal route.
$form_state->setRedirect('mymodule.page');
// Or
$form_state->setRedirect(
  'entity.node.canonical',
  ['node' => $node->id()]
);

// Redirect to a URI, or other web address
$url = Drupal\core\Url::fromUserInput('/some_page');
$form_state->setRedirectUrl($url);

// Redirect with params and options
$route_name = 'view.membership.membership';
$params = ['user' => $uid];
$options = [];
$form_state->setRedirect($route_name, $params, $options);

```

## EntityTypeManager (loadByProperties)

```
// return object
 $coupon = \Drupal::entityTypeManager()->getStorage('commerce_promotion_coupon')->load(3);
// ex: $coupon->usage_limit->value = 10; $coupon->save();
 $node = \Drupal::entityManager()->getStorage('node')->load($nid);
 
 // return array !!!
 $coupon = \Drupal::entityTypeManager()->getStorage('commerce_promotion_coupon')-> loadByProperties(['id' => '3']);
 // you need to pick from array
 $coupon = reset(\Drupal::entityTypeManager()->getStorage('commerce_promotion_coupon')-> loadByProperties(['id' => '3']));
``` 

## Using try / catch

```
try {
  // Some code here
}
catch (Exception $e) {
  // Generic exception handling if something else gets thrown.
  \Drupal::logger('widget')->error($e->getMessage());
}
``` 



# GET VALUE

## Get node type in template_preprocess_node

```
$node_type = $node->bundle();
``` 

## Get node type in template_preprocess_page

```
$node_type = $variables['node']->getType();
``` 

## Get webform submission !

```
$webform = \Drupal::entityTypeManager()->getStorage('webform')->load('my_form');
$webform = $webform->getSubmissionForm();
``` 

## Check permission current user

```
$user = \Drupal::currentUser();
$is_admin= $user->hasPermission('access administration pages');
```

## Get alias URL node

```
$alias = \Drupal::service('path.alias_manager')->getAliasByPath('/node/' . $nid);
``` 

## GET ENTITY VALUES

```
// List of fields that an entity has, the field definitions also have a lot of information like the type.
array_keys($entity->getFieldDefinitions())

// Use get() instead of the magic __get() on the entity level then you at least get some type hints.
$entity->get('field_name').

// Get the list of properties a certain field has, use array_keys() again for just the names, but the definitions also have the type and if it's computed or not.
$entity->getFieldDefinition('field_name')->getFieldStorageDefinition()->getPropertyDefinitions()

// Most field types have value property, but e.g. entity references have target_id and the computed entity. as you found. File and Image fields have additional properties like title/alt/description.
$entity->get('field_name')->value
$entity->get('field_name')->target_id
$entity->get('field_name')->entity

// Note that get('value') is not the same as ->value on the field item level, get() returns a typed data object, get('value')->getValue() is the same as ->value.

// When not specified, the delta 0 is assumed (all fields are a list internally, even something like the node id), you can use array access or the delta to access another delta, make sure it exists.
$entity->get('field_name')[1]->value
$entity->get('field_name')->get(1)->value

// When you have an entity reference, you can get the entity type and class like this:
$entity->get('field_name')->entity->getEntityTypeId()
$entity->get('field_name')->entity->getEntityType()->getClass()
// or 
get_class($entity->get('field_name')->entity)

// From there you can look up the interface and type hint against that, to a) make sure you have a valid, loadable reference and get type hints in an IDE:
$file = $entity->get('field_name')->entity;
if ($file instanceof \Drupal\file\FileInterface) {
  $file->getFileUri();
}
```

## Get values from referenced entity

```
foreach ($node->get('field_event')->referencedEntities() as $enty){

$entity = \Drupal::entityTypeManager()->getStorage("taxonomy_term")->load($enty->field_event_time->target_id);

$name = $entity->getName();
print_r($name);

}
``` 

## Get Nodes, Many Ways (load, route, php, from db)

```
// Load a node
$listing_page_node = \Drupal\node\Entity\Node::load($node_id);

// Get current node from the route
\Drupal::routeMatch()->getParameter('node')
\Drupal::routeMatch()->getParameter('node')->getType()

// Get node in PHP and get field
if(\Drupal::routeMatch()->getParameters()->has('node')) {
    $current_node = \Drupal::routeMatch()->getParameter('node');
    if($current_node->getType() === 'campaign_page') {
        $variables['show_limited_menu'] = $current_node->get('field_show_limited_menu')->value;
    }
}

// Get nodes from database, sort, filter and prepare display mode render array
$query = \Drupal::entityQuery('node')
                ->condition('type', 'product_service_detail')
                ->condition('status', 1)
                ->sort('created', 'DESC')
                ->range(0, 4);
                $nids = $query->execute();
                $nodes = entity_load_multiple('node', $nids);
                $variables['featured_nodes'] = \Drupal::entityTypeManager()->getViewBuilder('node')->viewMultiple($nodes, 'card');
``` 

# FORM 

## Get FORM

```
$form = \Drupal::formBuilder()->getForm('Drupal\user\Form\UserLoginForm');
``` 

## Form redirection on submit

```
public function submitForm(array &$form, FormStateInterface $form_state) {
  $form_state->setRedirect('user.page');
}
``` 

# Render a Render array to HTML code

```
$result =  array(
  '#markup' => 'Hello. This is my First Page',
);
$renderer = \Drupal::service('renderer');
$html = $renderer->render($result);
```

# Pass arguments to buildForm()
```
// in preprocessing.php
$variables['lead_form'] = \Drupal::formBuilder()->getForm('Drupal\my_form_module\Form\LeadForm', 'hero_video');

//it passes that extra variable through getForm into an arguments array added on the end of buildForm
    
public function buildForm(array $form, FormStateInterface $form_state, string $gated_content_context = NULL) {
    if($gated_content_context === 'hero_video') { ... }
    ....
}
```

----------------------------------------------------


# NODE and ENTITY

# Get field value of a Node / Entity

```
$value = $node->get($field)->value;
$target = $node->get($field)->target_id; // For entity ref (Term and etc)

// Get / Render image (Show current user's picture)
$userCurrent = \Drupal::currentUser();
$user = \Drupal\user\Entity\User::load($userCurrent->id());
$renderd_image = $user->get('user_picture')->first()->view();
```

# Get multiple values of a Node / Entity

```
foreach ($node->get('field_images')->getValue() as $key => $image) {
    $image = $node->field_images[$key]->view($settings);
}
//Example Of $settings
$settings = ['settings' => ['image_style' => 'thumbnail']];
```

# Get Entity data and metadata

```
$nodeEntity = \Drupal::entityTypeManager()->getDefinition('node');
```

## Create a node programmatically (Article)

```
$language = \Drupal::languageManager()->getCurrentLanguage()->getId();
$node = \Drupal\node\Entity\Node::create(array(
          'type' => 'article',
          'title' => 'The title',
          'langcode' => $language,
          'uid' => 1,
          'status' => 1,
          'body' => array('The body text'),
          'field_date' => array("2000-01-30"),
            //'field_fields' => array('Custom values'), // Add your custon field values like this
    ));
$node->save();
``` 

## Update a node / Entity

```
// Update title
$node = \Drupal\node\Entity\Node::load($nid);
$node->setTitle('The new Title')
$node->save();

// Update a field ('body' and 'field_name')
$node = \Drupal\node\Entity\Node::load($nid);
$node->set("body", 'New body text');
$node->set("field_name", 'New value');
$node->save();

// To make changes after click on save / edit button, you can use the hook_entity_presave.
// hook_entity_presave(Drupal\Core\Entity\EntityInterface $entity);

function CUSTOM_MODULE_node_presave(Drupal\node\NodeInterface $node) {
  $node->setTitle('Edited Title');
  $node->set('body', 'this is the bew body');
  //CAUTION : Do not save here, because it's automatic.
}

// Using HOOK_entity_presave()

function mymodule_entity_presave(Drupal\Core\Entity\EntityInterface $entity) {
  if ($entity->getEntityType()->id() == 'node') {
    $entity->setTitle('The new Title');
    //CAUTION : Do not save here, because it's automatic.
  }
}
``` 



## Get node and nid from url

```
$node = \Drupal::routeMatch()->getParameter('node');
if ($node) {
  $nid = $node->id();
}
``` 

----------------------------------------------------


# USER

## Create a user account programmatically

```
$language = \Drupal::languageManager()->getCurrentLanguage()->getId();
$user = \Drupal\user\Entity\User::create();

//Mandatory settings
    $user->setPassword('password');
    $user->enforceIsNew();
    $user->setEmail('email');
    $user->setUsername('user_name');//This username must be unique and accept only a-Z,0-9, - _ @ .

//Optional settings
    $user->set("init", 'email');
    $user->set("langcode", $language);
    $user->set("preferred_langcode", $language);
    $user->set("preferred_admin_langcode", $language);
    //$user->set("setting_name", 'setting_value');
    $user->activate();

//Save user
    $res = $user->save();
```

## Get the current user

```
use Drupal\user\Entity\User;

$userCurrent = \Drupal::currentUser();
$user = \Drupal\user\Entity\User::load($userCurrent->id());
$name = $user->getUsername();
```

## Get user's roles

```
// Get All roles:
$roles = \Drupal\user\Entity\Role::loadMultiple();

// Get a user's roles:
$uid = 4; //The user ID
$user = \Drupal\user\Entity\User::load($uid);
$roles = $user->getRoles();

// To get current user's roles :
$userCurrent = \Drupal::currentUser();
$user = Drupal\user\Entity\User::load($userCurrent->id());
$roles = $user->getRoles();
```

----------------------------------------------------


# TAXONOMY

## Get taxonomy terms of a vocabulary

```
$query = \Drupal::entityQuery('taxonomy_term');
    $query->condition('vid', "tags");
    $tids = $query->execute();
    $terms = \Drupal\taxonomy\Entity\Term::loadMultiple($tids);
    
//Get Taxonomy name
$name = $term->toLink()->getText();

//Create link to the term
$link = \Drupal::l($term->toLink()->getText(), $term->toUrl());    
``` 

# Create taxonomy term programmatically

```
$term = \Drupal\taxonomy\Entity\Term::create([
          'vid' => 'test_vocabulary',
          'langcode' => 'en',
          'name' => 'My tag',
          'description' => [
            'value' => '<p>My description.</p>',
            'format' => 'full_html',
          ],
          'weight' => -1,
          'parent' => array(0),
    ]);
 $term->save();
 
 //Check Taxonomy name exist
    $query = \Drupal::entityQuery('taxonomy_term');
    $query->condition('vid', "test_vocabulary");
    $query->condition('name', "My tag");
    $tids = $query->execute();
``` 

# Show 1000 items on admin page

```
drush cset taxonomy.settings terms_per_page_admin 1000
``` 

----------------------------------------------------


# Domain Access

## Domain Access get all domains

```
// Load all domains
$storage = \Drupal::entityTypeManager()->getStorage('domain');
$domains = $storage->loadMultiple();

// Check active domain
$optionSelect = $domain->isActive() ? 'active"' : 'no-active';

// Domain HostName
$hostname = $domain->getHostname();

// Domain ID
$hostname = $domain->getDomainId();

// Domain name or path
$name = $domain->get('name');
$fullUrl = $domain->get('path');

// Get the user IP address
$ip = \Drupal::request()->getClientIp();
```

----------------------------------------------------

# URL

## Get the current page URI or FRONT

```
$current_path = \Drupal::service('path.current')->getPath(); 
$current_uri = \Drupal::request()->getRequestUri(); 
$fullUrl = \Drupal::request()->getSchemeAndHttpHost() . $current_uri;

//Here an example to get the current page path programmatically in Drupal 8

$current_url = Url::fromRoute('<current>');
$path = $current_url->toString();

// Exemples for the page /en/user/login:
$current_url->toString(); // /en/user/login
$current_url->getInternalPath(); // user/login
$path = $current_url->getRouteName(); // <current>

// Get current route name
$route_name = \Drupal::routeMatch()->getRouteName();

// Get path alias
$current_path = \Drupal::service('path.current')->getPath();
\Drupal::service('path.alias_manager')->getAliasByPath($current_path);


// CHECK FRONT PAGE
$is_front_page = \Drupal::service('path.matcher')->isFrontPage();
```

## Get args from URL
```
$request = \Drupal::request();
$current_path = $request->getPathInfo();
$path_args = explode('/', $current_path);
$first_argument = $path_args[1];

// Example 2:
$current_url = Url::fromRoute('<current>');
$path = $current_url->toString();
//OR of you want to get the url without language prefix
$path = $current_url->getInternalPath();
$path_args = explode('/', $path);
```

## Create a link

```
use \Drupal\Core\Link;
$link = Link::fromTextAndUrl($text, $url);

// OR
$link = \Drupal\Core\Link::fromTextAndUrl($text, $url);
```

## Download a file from URL

```
$url = "https://site.com/filename.txt";
$result = system_retrieve_file($url, $destination = NULL, $managed = FALSE, $replace = FILE_EXISTS_REPLACE);
```

## CHECK FRONT AND GET CURRENT URL

```
use Drupal\Core\Url;

$output[]['#cache']['max-age'] = 0;
    if (\Drupal::service('path.matcher')->isFrontPage()) {
      $current_url = "";
    }
    else {
      $url = Url::fromRoute('<current>');
      $current_url = $url->toString();
    }
    $path = "http://" . \Drupal::request()->getHost() . $current_url;
```


## Get Path/Route from Url Object

```
use Drupal\Core\Url;

$url = Url::fromRoute('entity.node.canonical', ['node' => 1], []);
if ($url->isRouted()) {
  $out = $url->toString(); // Get : /en/node/1
  $out = $url->getInternalPath(); // Get : node/1
  $out = $url->getRouteName(); // Get : entity.node.canonical
  $out = $url->getRouteParameters(); // Get : Array ([node] => 1)
  $out = $url->getOptions(); // Get : Array ()
}
```
----------------------------------------------------

# FIELD

## Preprocess a Field, Change Theming of a field

```
// How to Change the Title, Change the description
/**
 * implements hook_element_info_alter()
 *
 */
function mymodule_alter_element_info_alter(&$type) {
  if (isset($type['date_popup'])) {
    $type['date_popup']['#process'][] = '_mymodule_date_popup_process_alter';
  }
}
function _mymodule_date_popup_process_alter(&$element, &$form_state, $context) {
  unset($element['date']['#description']);
  if ($element['#name'] == 'node_changed') {
    $element['date']['#title'] = "The Date is";
  }
  return $element;
}
```

## Add custom attributes to field

```
function netsoft_helpers_preprocess_field(&$variables) {
  if ($variables['field_type'] == 'link') {
  	if ($variables['field_name'] == 'field_website') {
      $variables['items']['0']['content']['#attributes']['target'] = '_blank';
      $variables['items']['0']['content']['#attributes']['rel'] = 'nofollow noopener';
  	}
  }
}
```

----------------------------------------------------

# VIEWS

## Alter a view results

```
function MYMODULE_views_pre_render(&$view) {
  if ($view->name == 'view_myviewname') {
    $result = $view->result;
    foreach ($result as $i => $row) {
      $view->result[$i]->field_field_myfieldtext[0]['rendered']['#markup'] = "The new text";
    }
  }
}

```

## Adding a condition to views with hook_views_query_alter ()

```
use Drupal\views\ViewExecutable;
use Drupal\views\Plugin\views\query\QueryPluginBase;

function MYMODULE_views_query_alter(ViewExecutable $view, QueryPluginBase $query) {
  if ($view->id() == 'loshad' && $view->getDisplay()->display['id'] == 'page_1') {
    $args = [
      ':now' => date('Y'),
      ':ages' => 4,
    ];
    $query->addWhereExpression('', 'node__field_god_rozhdeniya.field_god_rozhdeniya_value + :ages <= :now', $args);
  }
}

```

## Set views title programatically

```
function MYMODULE_views_pre_render(ViewExecutable $view) {

  if ($view->id() == 'catalog') {
    $city = 'Paris';
    $title = $view->getTitle() . ' ' . $city;
    $view->setTitle($title);
    $route = \Drupal::routeMatch()->getCurrentRouteMatch()->getRouteObject();
    $route->setDefault('_title', $title);
  }
}
```

## Views template suggestion

```
[base template name]--[view machine name]--[view display id].html.twig
[base template name]--[view machine name]--[view display type].html.twig
[base template name]--[view display type].html.twig
[base template name]--[view machine name].html.twig
[base template name].html.twig
```
----------------------------------------------------

# JS

## jquery once

```
$(".my-selector").once("comment_info").each(function() {
          alert(1);
});

$(".my-selector").once("test_fields").append("<div>hello world");

```

----------------------------------------------------

## Create a custom permission

```
// mymodule.permissions.yml
mymodule test_permission:
  title: 'My Test Permission'
  description: 'The description'
  restrict access: false
  
// mymodule.routing.yml
mymodule.home:
  path: 'test/home'
  defaults:
    _controller: '\Drupal\mymodule\Controller\Test::home'
    _title: 'My test Home'
  requirements:
    _permission: 'mymodule test_permission'
```

## How to read data from a url (good bye drupal_http_request() )

```
use GuzzleHttp\Exception\RequestException;
$url = "http://www.site.com";
$client = \Drupal::httpClient();
try {
  $response = $client->get($url);
  $data = $response->getBody();
  $code = $response->getStatusCode();
  $header = $response->getHeaders();
}
catch (RequestException $e) {
   watchdog_exception('my_module', $e);
}
```

## JSON Encode and Decode

```
use Drupal\Component\Serialization\Json;

$json = Json::encode($data);
$data = Json::decode($json);
```

## Export JSON PHP
```
$query = \Drupal::entityQuery('node');

$node_ids = $query
    ->condition('status', 1)
    ->exists('field_sidebar')
    ->execute();

$all_nodes_JSON = [];

foreach($node_ids as $node_id) {  
  $node = \Drupal\node\Entity\Node::load($node_id); 
  $sidebar_paragraphs = $node->get('field_sidebar')->referencedEntities();
  foreach($sidebar_paragraphs as $parag) {
      if($parag->getParagraphType()->id == 'related_links') {          
          $related_links = $parag->get('field_related_links')->referencedEntities();
          $related_nids = [];
          foreach($related_links as $linked_entity) {
            $related_node_JSON = [ 
              'id' =>  $linked_entity->id(),
              'url' => 'https://www.mysite.com/node/'.$linked_entity->id()
            ];
            array_push($related_nids, $related_node_JSON);
          }
          $this_node_JSON = [
            'nid' => $node_id,
            'title' => $node->getTitle(),
            'type' => $node->getType(),
            'url' => 'https://www.mysite.com/node/'.$node_id,
            'related_links' => $related_nids
          ];
          array_push($all_nodes_JSON, $this_node_JSON); 
          break;
      }
  }
}
$final_JSON = json_encode($all_nodes_JSON);
print_r($final_JSON);
```


## Create HTML Table

```
$header = ['#','Name', 'Mail'];
$data = [
  [1,'Name 1', 'Mail1@example.com'],
  [2,'Name N°2', 'second@example.com'],
];

$output[] = array(
  '#theme' => 'table',
  //'#cache' => ['disabled' => TRUE],
  '#caption' => 'The table caption / Title',
  '#header' => $header,
  '#rows' => $data,
);

// ADD CLASS

$header = ['#', 'Name', 'Mail'];
    $data = [
      [['data' => 2, 'class' => 'green'], 'Name 1', 'Mail1@example.com'],
      [['data' => 2, 'class' => 'red'], 'Name N°2', 'second@example.com'],
    ];
    
// ADD LINK to a table field
use Drupal\Component\Render\FormattableMarkup;

$value = new FormattableMarkup('<a href=":link">More</a>', [':link' => $url->toString()]);

// PREPROCESS
template_preprocess_table(&$vars);
```

# SUGGESTIONS

## Template Suggestions

```
//PAGE Suggestions
$suggestions[] = 'page__path_alias__' . formatPathAlias($path_alias);

//NODE Suggestions
$node = \Drupal::request()->attributes->get('node');
if (!is_null($node)) {
    $node_alias = \Drupal::service('path.alias_manager')->getAliasByPath('/node/'.$node->id());

    $view_mode = strtr($variables['elements']['#view_mode'], '.', '_');
    $suggestions[] = 'node__path_alias__' . formatPathAlias($path_alias) . '__' . $view_mode;
}

//BLOCK Suggestions
function mytheme_theme_suggestions_block_alter(array &$suggestions, array $variables) {
  // Block suggestions for custom block bundles.
  if (isset($variables['elements']['content']['#block_content'])) {
    array_splice($suggestions, 1, 0, 'block__bundle__' . $variables['elements']['content']['#block_content']->bundle());
  }
}


//FORM Suggestions
function mytheme_theme_suggestions_form_alter(&$suggestions, array $variables, $hook) {
  $suggestions[] = $hook . '__' . $variables['element']['#form_id'];
}

//INPUT Suggestions
$suggestions[] = $hook . '__' . $variables['element']['#form_id'] . '__' . $variables['element']['#type'];

//CONTAINER Suggestions
if (isset($variables['element']['#form_id'])) {
    $suggestions[] = $hook . '__' . $variables['element']['#form_id'];
}

//FORM ELEMENT Suggestions
if (isset($variables['element']['#name']) && isListingPageFacet($variables['element']['#name'])) {
    $suggestions[] = $hook . '__' . $variables['element']['#type'] . '__listing_page';
}

$suggestions[] = $hook . '__' . $variables['element']['#type'];

//FIELDSET Suggestions
if (isset($variables['element']['#name']) && isListingPageFacet($variables['element']['#name'])) {
    $suggestions[] = $hook . '__' . $variables['element']['#type'] . '__listing_page';
}

//Better Exposed Filters (BEF) Suggestions
if (isset($variables['element']['#name']) && isListingPageFacet($variables['element']['#name'])) {
    $suggestions[] = $hook . '__' . $variables['element']['#type'] . '__listing_page';
}
```
