# drupal8-snippets
Useful snippets

## How to get the current language

```
// To get the lanuage code:
$language = \Drupal::languageManager()->getCurrentLanguage()->getId();

//To get the language name:
$language =  \Drupal::languageManager()->getCurrentLanguage()->getName();
``` 

## Add a message to Drupal 8 Log system (good bye watchdog :))

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



# NODE

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
