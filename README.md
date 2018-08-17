# WP DB Schema Extender [![Build Status](https://travis-ci.org/LordDashMe/wordpress-db-schema-extender.svg?branch=master)](https://travis-ci.org/LordDashMe/wordpress-db-schema-extender) [![Coverage Status](https://coveralls.io/repos/github/LordDashMe/wordpress-db-schema-extender/badge.svg?branch=master)](https://coveralls.io/github/LordDashMe/wordpress-db-schema-extender?branch=master)

- A WordPress Database extender that provides a nice structure of table schema and data seeds.

## Requirement(s)

- PHP version from 5.6.* up to latest.

## Install

- It's advice to install the package via Composer. Use the command below to install the package:

```txt
composer require lorddashme/wordpress-db-schema-extender
```

## Usage

- You can start using the package without any configuration needed.

- Available functions:

| Function &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Description |
| -------- | ----------- |
| ```$schemaExtender->table('tableName', closure);``` | Use to create table structure. |
| ```$table->column('columnName', 'statement');``` | In the "table" function second argument closure you can use this function, this add the definition of you column. |
| ```$table->primaryKey('columnName');``` | Also the same with the "column" function you can use this function via "table" function  second argument closure, this add primary key to the table base on the given column name. |
| ```$schemaExtender->tableSeed('tableName', closure|array);``` | Use to seed data to the given table name. |
| ```$schemaExtender->tableName('tableName');``` | The return value of this function is concatenated with the wordpress prefix config. |
| ```$schemaExtender->rawQuery('statement');``` | Of course not all of the sql query is wrapped to php code that's why this function is provided to still allow you to do anything you want. |
| ```$schemaExtender->migrate();``` | Use to commit all the declared statement. |
| ```$schemaExtender->dropTable('tableName');``` | Use to drop the specified table. |
| ```$schemaExtender->dropTables(['tableName', ...]);``` | Use to drop tables in a single line of code. |

- Below are the sample implementation:

```php
<?php

include __DIR__  . '/vendor/autoload.php';

use LordDashMe\Wordpress\DB\SchemaExtender;

$schemaExtender = new SchemaExtender();

$schemaExtender->init(function($context) {

    $context->table('users', function($table) {
        $table->column('id', 'INT(11) NOT NULL AUTO_INCREMENT');
        $table->column('name', 'TEXT NULL');
        $table->primaryKey('id');
    });
    $context->table('user_options', function($table) {
        $table->column('id', 'INT(11) NOT NULL AUTO_INCREMENT');
        $table->column('user_id', 'INT(11) NOT NULL');
        $table->column('nick_name', 'INT(11) NOT NULL');
        $table->primaryKey('id');
    });
    $context->rawQuery('
        ALTER TABLE ' . $context->tableName('users') . '
            ADD KEY `user_id` (`user_id`);
        ALTER TABLE ' . $context->tableName('users_options') . ' 
            ADD CONSTRAINT `foreign_constraint_users_option_users` 
            FOREIGN KEY (`user_id`) 
            REFERENCES ' . $context->tableName('users') . ' (`id`) 
            ON DELETE CASCADE 
            ON UPDATE NO ACTION;'
    );
    $context->tableSeed('users', function($data) {
        $data->name = 'John Doe';
        return $data;
    });

});

$schemaExtender->tableSeed('user_options', function($data) {
    $data->user_id = 1;
    $data->nick_name = 'Nick Name' . rand();
    return $data;
})->iterate(2);

// You can attach the "migrate" function to "register_activation_hook" of wordpress.
// When the wordpress plugin set to active you can add the extender "migrate" function
// to execute all the query stored before the activation.
register_activation_hook( 
    '<wordpress>/wp-content/plugins/<your-plugin-name>/<your-plugin-name>.php', 
    function () use ($schemaExtender) {
        $schemaExtender->migrate();
    } 
);

```

- You can also use the SchemaExtender class like static-like class. See the "use" namespace path used.

```php
<?php

include __DIR__  . '/vendor/autoload.php';

use LordDashMe\Wordpress\DB\Facade\SchemaExtender;

SchemaExtender::init(function($context) {

    $context->table('users', function($table) {
        $table->column('id', 'INT(11) NOT NULL AUTO_INCREMENT');
        $table->column('name', 'TEXT NULL');
        $table->primaryKey('id');
    });
    $context->table('user_options', function($table) {
        $table->column('id', 'INT(11) NOT NULL AUTO_INCREMENT');
        $table->column('user_id', 'INT(11) NOT NULL');
        $table->column('nick_name', 'INT(11) NOT NULL');
        $table->primaryKey('id');
    });
    $context->rawQuery('
        ALTER TABLE ' . $context->tableName('users') . '
            ADD KEY `user_id` (`user_id`);
        ALTER TABLE ' . $context->tableName('users_options') . ' 
            ADD CONSTRAINT `foreign_constraint_users_option_users` 
            FOREIGN KEY (`user_id`) 
            REFERENCES ' . $context->tableName('users') . ' (`id`) 
            ON DELETE CASCADE 
            ON UPDATE NO ACTION;'
    );

});

SchemaExtender::tableSeed('users', function($data) {
    $data->name = 'John Doe';
    return $data;
});

SchemaExtender::tableSeed('user_options', function($data) {
    $data->user_id = 1;
    $data->nick_name = 'Nick Name' . rand();
    return $data;
})->iterate(2);

```

- The SchemaExtender class "tableSeed" function is not only for closure type argument, you can use array type if you want.

```php
<?php

include __DIR__  . '/vendor/autoload.php';

use LordDashMe\Wordpress\DB\SchemaExtender;

$schemaExtender = new SchemaExtender();
$schemaExtender->init();

$schemaExtender->tableSeed('users', [
    'name' => 'John Doe',
]);

$schemaExtender->tableSeed('user_options', [
    'user_id' => 1,
    'nick_name' => 'Nick Name' . rand()
])->iterate(2);

```

- The SchemaExtender class also provide a "dropTable" or "dropTables" function to accomodate the drop table action.

```php
<?php

include __DIR__  . '/vendor/autoload.php';

use LordDashMe\Wordpress\DB\SchemaExtender;

$schemaExtender = new SchemaExtender();
$schemaExtender->init();

$schemaExtender->dropTable('users');
$schemaExtender->dropTable('user_options');

// Or you can use the alis function that support multiple table names in one argument.
$schemaExtender->dropTables(['users', 'user_options']);

```

## License

- This package is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
