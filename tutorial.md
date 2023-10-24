## Структура на Laravel 
# Вовед

## Креирање на нов проект

## Структура на Laravel

Laravel
├── app
│   ├── Http
│   │   ├── **Controllers**
│   │   ├── **Middleware**
│   ├── Providers
│   ├── **Models**
│   ├── ...
├── bootstrap
├── config
├── database
│   ├── **migrations**
│   ├── seeders
│   ├── factories
│   ├── ...
├── **public**
├── resources
│   ├── views
│   ├── lang
│   ├── ...
├── routes
│   ├── **web.php**
│   ├── api.php
│   ├── ...
├── **storage**
│   ├── app
│   ├── framework
│   ├── logs
│   ├── ...
├── tests
├── vendor
├── **.env**
├── **.env.example**
├── .gitignore
├── composer.json
├── ...



# Model

## Migrations (Миграции)

### Создавање на нова миграција

За да креирате нова миграцијата ја користите следнава команда:

```shell
php artisan make:migration create_name_table
```

Фајлот ќе биде зачуван во `/app/database/migrations/`. 

### Структура на миграцијата

Миграциите се состојат од две главни методи `up` и `down`. 

Доколку сакаме да додадеме нов атрибут во табелата тоа го правиме во `public function up` така што го користиме параметарот $table->тип('име на атрибутот').

``` php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('post_votes', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }
    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('post_votes');
    }
};
```
Постојат многу податочни типови кои може да ги користиме при создавање, затоа ќе ги наброиме само најкористените:
- **Примарен клуч**: под defult е `$table->id()` но доколку сакаме да го променимнуваме во друго име, тогаш користиме `$table->primary('name')`.  
    - Доколку сакаме да направиме композитен примарен клуч, тогаш користиме: `$table->primary(['primary1','primary2'],..)`
- **Надворешен клуч**: `$table->foreign('name')->references('attribute')->on('tableName');`
  - **Cascade** и **restrict**: 
    - `->cascadeOnUpdate();`	- Направи cascade при update.
    - `->restrictOnUpdate();` -	Не дозволувај update.
    - `->cascadeOnDelete();` - 	Направи cascade при delete.
    - `->restrictOnDelete();` -	Не дозволувај бришење.
    - `->nullOnDelete();`	- При бришење, потолни ги подаотците со `null`.
- **Unique**: За да создадеме атрибут со единствени вредности користиме `->unique()`, на пример секој студент има единствен матичен број или индекс: `$table->string('index')->unique();`.
- **Defult**: Доколку сакаме да иницијалираме некоја вредност при креирање на нов запис користиме: `default(вредност)`. На пример `$table->boolean('online')->default(true)`.
- **Null**: Доколку сакаме некоја вредност да не ја дефинираме при создавањето на нов запис, тогаш користиме само `->nullable()`. На пример `$table->string('email')->nullable();`
- **Цел број**: `$table->integer('name');`
- **Булеан**: `$table->boolean('name');`
- **Текст**: `$table->string('name');` или `$table->text('name');`
- **Децимален** **број**: `$table->float('name', 8, 2);`


За да ја зачуваме миграцијата во базата користиме само `php artisan migrate` оваа команда ќе ја создаде новата табела со дефинираните атрибути. Кога ќе креираме нова миграција таа веднаш преоѓа во состобја `pending` па затоа оваа команда ќе ја зачува табелата. Но итсто така постојат и други команди кои треба да ги знаете:

| Команда                        | Опис                                                                                   |
| ------------------------------ | --------------------------------------------------------------------------------------------- |
|`php artisan migrate:status` | Ќе ги прикаже состојбите на сите миграции. |
| `php artisan migrate`          | Ќе ја изврши `up` методата на секоја миграција која е во состојба `pending`.|
| `php artisan migrate:refresh` | Доколку напраиме некаква промена на миграцијата и сакаме да се вратиме на иницијалната состојба, тогаш го користиме ова.                          |
| `php artisan migrate:fresh`    | Ќе ги избрише сите табели и повторно ќе ги создаде.       |
| `php artisan migrate:rollback` | Ќе ги негира последните миграцции кои сме ги направиле. |


Laravel користи obejct realtional mapper (ORM) кој ни дозволува работа со базата но преку php. 

За таа цел креираме модели кои ги поврзуавме со соодветните шеми кои ги дефиниравме во претходниот чекор.

Во следните чекори ќе објасниме како се поврзуваат миграциите и моделите и како може да работиме со нив.

## Создавање на нов модел

За да создадеме нов модел ја користиме командата: `php artisian make:model име`, на пример :

```
php artisan make:model Comment
php artisan make:model Post
```

Сите модели се сместени во `/app/Models`.

```php 
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Post extends Model
{
    // protected $table = 'posts';
}
```

---
**Како Laravel знае како да ги поврзе моделите и табелите во базата?**

Laravel пробува да ги спои табелите според имињата (доколку имаат слично име или заеднички зборови), но понекогаш имињата на моделите и табелите не ни се исти, па затоа најдобро е да ги поврземе сами со помош на командата: `protected $table = 'име на таблеата';`.

На пример доколку имаме некоја миграција со: `Schema::create('poinakvo_ime'` тогаш во моделот ќе може да ја споиме
 споиме со помош на:  `protected $table = 'poinakvo_ime';`. 

---


### Eloquent: Relationships

Во миграциите ние дефиниравме надворешни клучеви од една табела кон друга. 

Laravel ни нуди поедноставен и поефикасен начин за работа со тие врски. 

Битно е да знеате дека:

`belongsTo` и `belongsToMany` се користи кога овој модел ги рефренцира другите модели. `belongsTo` се користи кога имаме `one-to` додека пак `belongsTo` кога имаме `many-to`

`hasOne` или `hasMany` се користи од се користи кога овој модел е референциран од страна на друг модел. Кога имаме `one-to` се користи `hasOne`, кога имаме `many-to` се користи `hasMany`



#### one-to-one

На пример еден корисник има еден профил. Profile има надворешен клуч кон Student.

```php

class Student extends Model
{
    // името може да биде произволно
    public function profile()
    {
        // студент моделот е рефренциран од страна на Profile па затоа имаме hasOne
        return $this->hasOne(Profile::class);
    }
}

class Profile extends Model
{
    // името може да биде произволно
    public function stduent()
    {
        // има надворешен клуч кон Student
        return $this->belongsTo(Student::class);
    }
}
```

#### one-to-many

На пример една објава има многу коментари. 

```php
class Post extends Model
{
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}

class Comment extends Model
{
    public function post()
    {
        return $this->belongsTo(Post::class);
    }
}
```



####  many-to-many

Еден студент може да има многу предмети и еден предмет може да има многу студенти.

```php
class Course extends Model
{
    public function enroll()
    {
        return $this->hasMany(Enroll::class);
    }
}

class Enroll extends Model
{
    public function courses()
    {
        return $this->belongsToMany(Course::class);
    }

    public function students()
    {
        return $this->belongsToMany(Student::class);
    }


}

class Student extends Model
{
    public function enroll()
    {
        return $this->hasMany(Enroll::class);
    }
}
```

Овој начин ни дава поголема контрола врз кодот, бидејќи имаме модел `Enroll` со кој полесно ќе работиме.

Вториот начин е да ги поврземе `Student` и `Course` директно, овај начин ќе ни генерира нова **пивот** табела во базата со која може да работиме. 


```php
class Student extends Model
{
    public function courses()
    {
        return $this->belongsToMany(Course::class);
    }
}

class Course extends Model
{
    public function students()
    {
        return $this->belongsToMany(Student::class);
    }
}

```

За да ги споиме студентите и курсевите, ние ќе мора да користиме `attach()`, `detach()`.

##### Attach/Detach 

Со помош на `detach()` ја острануваме врската за одреден објект, додека пак `attach()` ќе креира нова врска.

Пример:

Да го земеме претходниот пример со студенти и курсеви:

Студенти:

| ID | Име     | Презиме   |
|----|---------|-----------|
| 1  | Петар   | Петровски |
| 2  | Ана     | Андоновска|
| 3  | Марко   | Марковски |
| 4  | Елена   | Георгиева |
| 5  | Иван    | Ивановски |

Курсеви:

| ID | Курс                     |
|----|--------------------------|
| 1  | Дискретна Математика     |
| 2  | Структурно Програмирање  |


Доколку сакаме да ги поврземе курсеви и студентите, тоа мора да го направиме преку `attach()`, моментално пивот табелата е празна.


```php
// најди го студентот со ID = 2 (Ана)
$student = Student::find(2); 

// најди го курсот со ID = 1 (Структурно Програмирање)
$course = Course::find(1);
// во пивот табелата додади ја оваа врска
$student->courses()->attach($course); 

// истито може да се направи и обратно:
// $course->students()->attach($student); 

```

Доколку сакаме да додеме низа од курсеви или студенти, го правиме тоа преку нивните ID's

```php
// најди го студентот со ID = 5 (Иван)
$student = Student::find(5);
// низа со IDs на курсевите (Дискретна Математика со ID = 1 и Структурно Програмирање со ID = 2) 
$courseId = [1, 2]; 

// во пивот табелата додади ја оваа врска
$student->courses()->attach($courseIds); 
```

По извршување на наредбите, пивот табелата ќе ги има следниве податоци:

| student_id | course_id |
|-----------|----------|
| 2         | 1        |
| 5         | 1        |
| 5         | 2        |

За да избришеме некоја врска користиме `detach()`, работи на истиот начин како и `attach()`.

```php
//најди го курсот со ID = 1 (Структурно Програмирање)
$course = Course::find(1);
// најди го студентот со ID = 5 (Иван)
$student = Student::find(5); 

// избриши ја оваа врска од пивот табелата
$course->students()->detach($student); .

// $course->students()->detach([1,5]); 
```

По извршување на наредбата, пивот табелата ќе ја избрише дефинираната врска од табелата.


| student_id | course_id |
|-----------|----------|
| 2         | 1        |
| 5         | 2        |


## Работа со модели 

Да речеме дека го имаме модел `Student` со атрибути `id`, `име`, `презиме`:

| ID | Име     | Презиме   |
|----|---------|-----------|

### Create

Во Laravel, можете да креирате нов запис во моделот Student на неколку начини:

Користејќи го `new` клучниот збор и методот `save`:

```php
$student = Student();

$student->име = 'Петар';
$student->презиме = 'Петровски';

$student->save();
```

Користејќи го `create`:

```php
Student::create([
    'име' => 'Ана',
    'презиме' => 'Андоновска'
]);
```

За да може да работи `create` мора во моделот да додадеме `protected $fillable` и да ги наброите сите атрибути кои сакаме да ги користиме при креирање. во случајот користиме само `име` и `презиме`, па затоа:

```php
class Student extends Model
{
    // доколку имате другит атрибути мора да ги додадете tuka за да работи create методот.
    protected $fillable = ['име', 'презиме'];
}
```

Ова не важи за `save`. 

| ID | Име     | Презиме   |
|----|---------|-----------|
| 1  | Петар   | Петровски |
| 2  | Ана     | Андоновска|


### Update

Во Laravel, можете да ажурирате некој запис на неколку начини:

Со помош на `save`, прво го наоѓаме студентот со помош на методот `find` па потоа правиме некаква промена и истата ја зачувуваме.

```php
// најди го студентот со ID = 1 (Петар)
$student = Student::find(1);
//смени го името и презимето на овој студент
$student->име = 'Марко';
$student->презиме = 'Маркоски';
// зачувај ја промената
$student->save();
```

| ID | Име     | Презиме   |
|----|---------|-----------|
| 1  | Марко   | Маркоски  |
| 2  | Ана     | Андоновска|

Вториот начин е со користење на методот `update`.

```php
// најди го студнетот со ID = 1 (Марко)
$student = Student::find(1);

$student::update([
    'име' => 'Петар'
    'презиме' => 'Петровски'
]);
```

За да може да работи `update` мора да ги имаме сите атрибути дефинирано во `fillable`, исто како во `create`.

### Delete

Бришење на запис од базата може да се направи на неколку начини, со помош на `delete` или `destroy`. 

`delete` го брише записот за одреден елемент, додека пак `destory` ги брише записите според примарниот клуч

```php
$student = Student::find(1);
$student.delete();

// Student::destory(1);
// Student::destory([1,2]);
```

### one-to-one, one-to-many

Како што може да забележите за да се креира `one-to-one` или `one-to-many` врска доволно е само да го дефинираме надворешниот клуч и да му зададеме вредност. 

```php
$profile = new StudentProfile();
$profile->student_id = 1;
$profile->save();
```
Но Laravel ни нуди полесен начин на поврузавње на ваквите типови на врски.

На пример:

Ја имаме табелата `Student`:

| ID | Име     | Презиме   |
|----|---------|-----------|
| 1  | Марко   | Маркоски  |
| 2  | Ана     | Андоновска|

И табелата `Profile`:

| ID | Пол     | student_id|
|----|---------|-----------|
| 1  | М       | 1         |

И ги имаме следниве модели:

```php
class Student extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}

class Profile extends Model
{
    public function student()
    {
        return $this->belongsTo(Student::class);
    }
}
```

Како што може да видите имаме `one-to-one` врска помеѓу `Student` и `Profile`. `Profile` чува надворешен клуч кон `Student`.

Доколку сакаме да создадеме нов `Profile` при креирањето на `Student` може тоа да го направиме на овој начин:

```php
// наместо find, може да креирате и нов студент
$student = Student::find(2);
$profile = new StudentProfile();
// на овој начин се доделуваат атрибути
$profile->пол('Ж');
$student->profile()->save($profile);
```

Со помош на овој код, Laravel ќе ни креира нов запис во табелата `Profile` и во `student_id` ќе го додаде својот `id`. 

По извршување на кодот табелата ќе изгледа вака:

| ID | Пол     | student_id|
|----|---------|-----------|
| 1  | M       | 1         |
| 2  | Ж       | 2         |

Овој начин е многу поефикасен и полесен.  

# Controller

Задачата на еден контролер во Laravel е да преработи некое HTTP барање и да врати некаков `view` или `податоци`, во овој дел ќе го објасниме тоа.

## Создавање на нов контролер

За да создадеме нов контролер ние ја користиме командата:

`php artisan make:controller ИмеController`

Контролерите се зачувани во: `/app/Http/Controllers`.

### Работа со контролер

Доколку отворите некој контролер, може да приметите дека Laravel сам ни генерира неколку методи, ние може да ги користиме тие или да си креираме сопствени методи. 

Во овој дел ќе научиме како да работиме со барањата и како да вратиме `view` или податоци од базата.

#### Request

`Request` е библиотека која ни овозможува работа со HTTP барања, за да може да ја користиме оваа библиотека, само го додаваме `Request $request` како параметар во методот.

Со помош на `$request` ние може да пристапиме до сите испратени податоци: file, integer, string, ip, header.

Исто така може да направиме и валидација на податоците користејќи `$request->validate`. 


На пример сакаме да ги испратиме следниве податоци:

```
{
    "name": "Ema",
    "integer": 12,
    "float": 3.14,
    "boolean": true,
    "custom-header": "CustomHeaderValue",
    "nestedArray": {
        "key1": 32,
        "key2": 33
    },
    "file": "cat.jpg"
}

```

Во контролерот:

```php
<?php
namespace App\Http\Controllers;

use App\Models\Example;
use Illuminate\Http\Request;

class ExampleController extends Controller
{
    public function create(Request $request)
    {
        // Валидација на податоци
        // доколку не исполниме некој услов од валидацијата, нема да се изврши останатиот код и ќе ни врати само ерор.
        // required значи дека мора да е испратен тој податок
        $request->validate([
            'name' => 'required|max:255', // да нема над 255 карактери
            'integer' => 'required|integer',  // цел број
            'float' => 'required|numeric', // број
            'boolean' => 'required|boolean', // булеан
            'file' => 'file', // фајл
            'custom-header' => 'string', // текст
        ]);


        // со помош на input ние може да ги земеме сите податоци на една форма: текст,
        // цели броеви, децимални броеви, булеан вредности. 
        $name = $request->input('name'); // Ema
        $integer = $request->input('integer'); // 12
        $float = $request->input('float'); // 3.14
        $boolean = $request->input('boolean'); // true

        $nestedArray = $request->input('nestedArray'); //nestedArray

        // Како да пристапиме до вредностите
        $key1 = $nestedArray['key1']; // 32 
        $key2 = $nestedArray['key2']; // 33

        // фајлови, прво проверуваме дали е испратен фајлот со помош на hasFile
        if ($request->hasFile('file')) {
             $file = $request->file('file'); // cat.jpg
        }

        // header кој бил испратен
        $header = $request->header('custom-header'); //CustomHeaderValue

        // IP адреса на испраќачот
        $ip = $request->ip();

        Example::create({
            'name': $name,
            'integer': $integer,
            'ip': $ip,
            ...
        });
    }
}
```

#### Работа со фајлови





    

## Рути

Во Lravel, `web.php` е задолжен за хендлање сите HTTP барања, `web.php` се наоѓа во `/app/routes`

Со помош на библиотеката (фасада) `Route` ние ќе може да ги обработиме сите барања.

### Создавање на нова рута

Доколку сакате барањето да ви биде обработено од страна на некој контролер се користи ова:

`Route::тип('url', 'Controller@method');`


`Route::тип('url', [Controller::class, 'method']);
`

- `тип` може да биде: `GET`, `POST`, `DELETE`, `PUT`.
- `url` е патеката, на пример `/` или `/home`
- `Controller` некој контролер.
- `method` е метод во контролерот.

Пример:

```php
Route::get('/create', [PostController::class, 'create']);
Route::post('/create', [PostController::class, 'store']);
```

Доколку сакате да вратите нешто директно од `web.php` без користење на контролер тогаш може да напишете функција на овој начин:

```php
Route::get('/', function () {
    return 'Return something';
});
```

Исто така може и да групираме рути со заеднички префикс, на пример доколку имаме `/post/create` и `/post/delete`, можеме да ги групираме со користење на `Route::prefix('')->group(function({}))`:

```php
Route::prefix('post')->group(function () {
   Route::post('/create', [PostController::class, 'create']);
   Route::post('/delete', [PostController::class, 'delete']);
});   
```


