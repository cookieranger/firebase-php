##############
Authentication
##############

In order to access a Firebase project using a server SDK, you must authenticate your server with Firebase. This
can be done either by creating and using a
`Service Account <https://developers.google.com/identity/protocols/OAuth2ServiceAccount>`_, or by providing using a
database secret.

.. note::
    Service Account authentication has been introduced with Version 3 of the Firebase platform, and
    authenticating with a database secret is to be considered a legacy method. This SDK provides both
    methods, but be aware that the database secret method only works with the Realtime Database. As soon
    as you want to access Firebase Storage, you will have to use a Service Account, and although the
    Firebase team has not announced an End Of Life of database secret authentication, the recommendation
    is to use the Service Account authentication.

*****************************
With a Google Service Account
*****************************

Follow the steps described in the official Firebase documentation to create a Service Account for your Firebase
application: `Add Firebase to your app <https://firebase.google.com/docs/server/setup#add_firebase_to_your_app>`_.

You can now create a new Firebase instance with ``Firebase::fromServiceAccount($value)`` which accepts one of the
following values:

- the path to a Google Service Account JSON configuration file (recommended)
- a JSON string
- an array
- an instance of ``Firebase\ServiceAccount``

.. code-block:: php

    $firebase = Firebase::fromServiceAccount(__DIR__.'/google-service-account.json');

If the project ID in the JSON file does not match the URL of your Firebase application, or if you want to
be explicit, you can specify the Database URI either as a second parameter or an additional method call:

.. code-block:: php

    $firebase = Firebase::fromServiceAccount(
        __DIR__.'/google-service-account.json',
        'https://my-project.firebaseio.com'
    );

    $firebase = Firebase::fromServiceAccount(__DIR__.'/google-service-account.json');
        ->withDatabaseUri('https://my-project.firebaseio.com');


**********************
With a Database secret
**********************

You can create and retrieve Database secrets when you navigation to **Project Settings**/**Database**.

.. code-block:: php

    $secret = '...';
    $databaseUri = 'https://my-project-id.firebaseio.com';

    $firebase = Firebase::fromDatabaseUriAndSecret($databaseUri, $secret);

.. note::
    This is a legacy authentication method, you will only be able to access the Firebase Realtime Database
    when using it. If you want to access the Storage or other parts of your Firebase project, you will
    have to use Service account authentication.

***********************************************
Authentication overrides (a.k.a. Custom Tokens)
***********************************************

You can impersonate users of your Firebase application through the the ``asUserWithClaims()`` method, which requires
a user id as the first parameter, and an optional array with claims as the second.

.. code-block:: php

    $firebase = Firebase::fromServiceAccount(...);

    $authenticated = $firebase->asUserWithClaims('a-user-id', [
        'premium-user' => true
    ]);

If you want to be more explicit, you can also override the authentication just on a database connection:

.. code-block:: php

    $firebase = Firebase::fromServiceAccount(...);
    $database = $firebase->getDatabase();

    $authenticated = $database->asUserWithClaims('a-user-id', [
        'premium-user' => true
    ]);

.. note::
    Under the hood, the SDK creates a
    `Custom Token <https://firebase.google.com/docs/auth/server/create-custom-tokens>`_ and uses to apply
    the `Security rules <https://firebase.google.com/docs/database/security/>`_ to the connection.

    Authentication overrides are performed differently, depending on whether you authenticate with a
    Google Service Account or a database secret.
