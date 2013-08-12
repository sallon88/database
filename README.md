Config
---------
	$DB_CONFIG = array(
		'default' => array(
			'host' => '127.0.0.1',
			'database' => 'test',
			'username' => 'root',
			'password' => '',
		),

		'test' => array(
			'host' => '192.168.0.224',
			'database' => 'msg',
			'username' => 'root',
			'password' => '',
			'port' => '3306',
			'charset' => 'utf8',
			'prefix' => '',
			'cache_path' => __DIR__.'/cache/test' // enable cache table structrue by give a cache path
		),
	);

Connection
----------------
connect to a database and open a table, 

	DB::connection('default')->table('users');

since it`s a default connection, the above clause can be written as:
 
	DB::table('users')

connect to default database and execute a raw query

	DB::query('select * from users where user_id = ?', array(1))
	
last information

	DB::lastInsertId();
	DB::lastSql();

Query
-----------
###where
the only parameter of select method is $where, note that there are 9 different possible forms. 

	DB::table('users')->select();   // no where
	DB::table('users')->select(1);  // WHERE user_id = ?
	DB::table('users')->select('user_id = 1'); // WHERE user_id = 1
	DB::table('users')->select(array(1,2,3)); // WHERE user_id IN (?,?,?)

	DB::table('users')->select(array(
		'user_name' => 'user_name'
	)); 
	// WHERE user_name = ?
	
	DB::table('users')->select(array(
		'user_name' => '%user_name%'
	))
	// WHERE user_name LIKE ?

	DB::table('users')->select(array(
		'user_name' => array('name1', 'name2', 'name3')
	)) 
	// WHERE user_name IN (?,?,?)
	
	DB::table('users')->select(array(
		'age >' => 30, 
		'status not in' => array(1, 2, 3)
	));
	// WHERE (age > ?) AND (status NOT IN (?,?,?))

	DB::table('users')->select(array(
		array('age >' => 30), 
		array('user_name' => '%aa%', 'sex' => 1)
	));
	// WHERE (age > ?) OR ((user_name LIKE ?) AND (sex = ?))

###chain options

	DB::table('users')->field('user_name as name, age, sex')->selectOne();
	DB::table('users')->order('name desc, age asc')->select();
	DB::table('users')->limit('5, 10')->select();
	DB::table('users')->distinct(1)->select();
	DB::table('users')->group('age')->total();


###table modify

	DB::table('users')->update(array('user_name' => 'name'), $where);
	DB::table('users')->insert(array('user_name' => 'name'));

	// where in delete is same as in select, except that, when $where is null, select will select all, delete will delete nothing
	DB::table('users')->delete($where);

### transaction

	DB::transaction(function(){
		DB::table('users')->insert();
		DB::table('orders')->update();
	});

Model
=======

	class User extends Model {
		protected static $table_name = 'users';

		protected static $validates = array(
			'user_name' => 'required, length:6:20, alpha_begin, alpha_dash',
			'email' => 'required, email',
			'age' => 'integer'
		);
	}

by extends the Model class, User will have all the instance methods of DB::table('users') as its static methods

	User::select($where); // DB::table('users')->select($where)
	User::delete($where); // DB::table('users')->delete($where)

and 3 dynamic methods which are getBy*, getOneBy*, deleteBy*.
Note that given a table field 'user_name', the according dynamic method name shall be exactly 'getByUserName', case sensitive!

	User::getByUserName('%aa%');
   	//DB::table('users')->select(array('user_name' => '%aa%'));
	
	User::getByUserNameAndSex('%aa%', '1');
   	//DB::table('users')->select(array('user_name' => '%aa%', 'sex' => 1));
	
	User::deleteByUserNameAndSexAndAge('%aa%', '1', array(15, 20, 25));
	//DB::table('users')->delete(array('user_name' => '%aa%', 'sex' => '1', 'age' => array(15, 20, 25)));

params in create and update method will be automatically checked according to $validates

	User::create(array(
		'user_name' => 'user_name',
		'email' => 'a@a.com'
	));
	// create will succeed since age is not required

	User::update(array(
		'user_name' => 'user_name',
		'age' => 'abc'
	), $where);
	// update will fail since age is not a valid integer
