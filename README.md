# drupal8-snippets
Useful snippets

## How to get the current language

```
// To get the lanuage code:
$language = \Drupal::languageManager()->getCurrentLanguage()->getId();

//To get the language name:
$language =  \Drupal::languageManager()->getCurrentLanguage()->getName();
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
