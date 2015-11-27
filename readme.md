# attribute-manipulation
Allows multiple traits to manipulate with the attributes of an Eloquent Model.

## Installing

Install using Composer `composer require larapack/attribute-manipulation 1.*`.

## Why?

Some traits might want to manipulate with attributes of an Eloquent Model, this can however be done easy creating the `getAttribute`-method and/or `setAttribute`-method.
However if multiple traits uses those methods then they will complain about eachother since it is only allow for a class to use one trait with the same method.
So if you are up to having a multiple traits using the same methods to manipulate the Eloquent Model attributes then you might use our methods instead for better support.

## Trait setup

Here is how a trait using our methods should look like:

```
<?php

namespace Vendor\Package;

use Exception;
use Larapack\AttributeManipulation\Manipulateable;

trait ExampleTrait
{
    /**
     * Boot the ExampleTrait trait for a model.
     * @return void
     */
    public static function bootExampleTrait()
    {
        // first we will check if the model uses the Manipulateable trait, if not we will throw them an friendly error
        if (!isset(class_uses(get_called_class())[Manipulateable::class])) {
            throw new Exception(sprintf('You must use the '.Manipulateable::class.' trait in %s to use the ExampleTrait trait.', get_called_class()));
        }

        // then we will set our manipulator for the attribute setter
        static::addSetterManipulator(function($model, $key, $value) {
            // here we have both the model object ($model), attribute key ($key) and the current value to set ($value).
            // whatever we return is that value that will be sent to the next manipulator 
            // or if none then it will set set to the model object.
            // For this example we will return the encrypted value.
            // Example: $model->password = 'secret' will be saved as the encrypted value of `secret`.
            // For the real encryptable trait which this example is based on please view https://github.com/larapack/attribute-encryption
            return \Crypt::encrypt($value);
        });

        // then we will set our manipulator for the attribute getter
        static::addGetterManipulator(function($model, $key, $value) {
            // here we have both the model object ($model), attribute key ($key) and the current value to set ($value).
            // whatever we return is that value that will be sent to the next manipulator 
            // or if none then it will returned to the output.
            // For this example we will return the decrypted value.
            // Example: echo $model->password will output the decrypted value of `password`.
            // For the real encryptable trait which this example is based on please view https://github.com/larapack/attribute-encryption
            return \Crypt::decrypt($value);
        });
    }
}
```

Example model:
```
<?php

namespace App;

use Larapack\AttributeManipulation\Manipulateable;
use Vendor\Package\ExampleTrait;

class User
{
    use Manipulateable;
    use ExampleTrait;

    //...
}
```

Example test:
```
$user = new App\User;
$user->password = 'secret';
dump($user); // here you will see that the password is encrypted
echo $user->password; // this will output the decrypted password
```
