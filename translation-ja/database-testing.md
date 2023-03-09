# データベーステスト

- [イントロダクション](#introduction)
    - [各テスト後のデータベースリセット](#resetting-the-database-after-each-test)
- [モデルファクトリ](#model-factories)
- [シーダの実行](#running-seeders)
- [利用可能なアサート](#available-assertions)

<a name="introduction"></a>
## イントロダクション

データベース駆動型アプリケーションのテストを容易にするため、Laravelはさまざまな便利なツールとアサートを提供しています。それに加えて、Laravelモデルのファクトリ（テスト用インスタンスの生成)とシーダ（初期データ設定)により、アプリケーションのEloquentモデルとリレーションを使用し、テストデータベースレコードを簡単に作成できます。これらの強力な機能のすべてについて、以降のドキュメントで説明します。

<a name="resetting-the-database-after-each-test"></a>
### 各テスト後のデータベースリセット

先へ進む前に、以前のテストデータが後続のテストに干渉しないように、各テストの後にデータベースをリセットする方法について説明しましょう。Laravelに含まれている`Illuminate\Foundation\Testing\RefreshDatabase`トレイトがこれを処理します。テストクラスでトレイトを使用するだけです。

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * 基本的な機能テスト例
         */
        public function test_basic_example(): void
        {
            $response = $this->get('/');

            // ...
        }
    }

`Illuminate\Foundation\Testing\RefreshDatabase`トレイトは、スキーマが最新であれば、データベースをマイグレートしません。その代わりに、データベーストランザクション内でテストを実行するだけです。したがって、このトレイトを使用しないテストケースによってデータベースに追加されたレコードは、まだデータベースに残っている可能性があります。

データベースを完全にリセットしたい場合は、代わりに`Illuminate\Foundation\Testing\DatabaseMigrations`、または`Illuminate\Foundation\Testing\DatabaseTruncation`トレイトを使用してください。しかし、両選択肢とも、`RefreshDatabase`トレイトよりもかなり遅いです。

<a name="model-factories"></a>
## モデルファクトリ

テスト時、テストを実行する前にデータベースへレコードを挿入する必要がある場合があると思います。このようなテストデータ作成時、各カラムの値を手作業で指定する代わりに、Laravelでは[モデルファクトリ](/docs/{{version}}/eloquent-factories)を使用し、デフォルトの属性セットを各Eloquentモデルに対して定義できます。

モデルを生成するモデルファクトリの作成と利用の詳細は、完全な[モデルファクトリドキュメント](/docs/{{version}}/eloquent-factories)を参照してください。モデルファクトリを定義したら、テスト内でそのファクトリを利用してモデルを作成できます。

    use App\Models\User;

    public function test_models_can_be_instantiated(): void
    {
        $user = User::factory()->create();

        // ...
    }

<a name="running-seeders"></a>
## シーダの実行

機能テスト中に[データベースシーダ（初期値設定）](/docs/{{version}}/seeding)を使用してデータベースへデータを入力する場合は、`seed`メソッドを呼び出してください。`seed`メソッドはデフォルトで、`DatabaseSeeder`を実行します。これにより、他のすべてのシーダが実行されます。または、特定のシーダクラス名を`seed`メソッドに渡します。

    <?php

    namespace Tests\Feature;

    use Database\Seeders\OrderStatusSeeder;
    use Database\Seeders\TransactionStatusSeeder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * 新しい注文の作成をテスト
         */
        public function test_orders_can_be_created(): void
        {
            // DatabaseSeederを実行
            $this->seed();

            // 特定のシーダを実行
            $this->seed(OrderStatusSeeder::class);

            // ...

            // 配列中に指定してあるシーダを実行
            $this->seed([
                OrderStatusSeeder::class,
                TransactionStatusSeeder::class,
                // ...
            ]);
        }
    }

あるいは、`RefreshDatabase`トレイトを使用する各テストの前に、自動的にデータベースをシードするようにLaravelに指示することもできます。これを実現するには、テストの基本クラスに`$seed`プロパティを定義します。

    <?php

    namespace Tests;

    use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

    abstract class TestCase extends BaseTestCase
    {
        use CreatesApplication;

        /**
         * デフォルトのシーダーが各テストの前に実行するかを示す
         *
         * @var bool
         */
        protected $seed = true;
    }

`$seed`プロパティが `true` の場合、`RefreshDatabase`トレイトを使用する各テストの前に`Database\Seeders\DatabaseSeeder`クラスを実行します。ただし，テストクラスに`$seeder`プロパティを定義し，実行したい特定のシーダーを指定できます。

    use Database\Seeders\OrderStatusSeeder;

    /**
     * 各テストの前に特定のシーダーを実行
     *
     * @var string
     */
    protected $seeder = OrderStatusSeeder::class;

<a name="available-assertions"></a>
## 利用可能なアサート

Laravelは、[PHPUnit](https://phpunit.de/)機能テスト用にいくつかのデータベースアサートを提供しています。以下にこのようなアサートについて説明します。

<a name="assert-database-count"></a>
#### assertDatabaseCount

データベース内のテーブルに指定した数のレコードが含まれていることをアサートします。

    $this->assertDatabaseCount('users', 5);

<a name="assert-database-has"></a>
#### assertDatabaseHas

データベース内のテーブルに、指定したキー／値クエリの制約に一致するレコードが含まれていることをアサートします。

    $this->assertDatabaseHas('users', [
        'email' => 'sally@example.com',
    ]);

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

データベース内のテーブルに、指定したキー／値クエリの制約に一致するレコードが含まれていないことをアサートします。

    $this->assertDatabaseMissing('users', [
        'email' => 'sally@example.com',
    ]);

<a name="assert-deleted"></a>
#### assertSoftDeleted

`assertSoftDeleted`メソッドは、指定したEloquentモデルが「ソフトデリート」されたことをアサートします。

    $this->assertSoftDeleted($user);

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

`assertNotSoftDeleted`メソッドは、指定Eloquentモデルが、「ソフトデリート」されていないことをアサートします。

    $this->assertNotSoftDeleted($user);

<a name="assert-model-exists"></a>
#### assertModelExists

指定モデルがデータベースに存在することをアサートします。

    use App\Models\User;

    $user = User::factory()->create();

    $this->assertModelExists($user);

<a name="assert-model-missing"></a>
#### assertModelMissing

指定モデルがデータベースに存在しないことをアサートします。

    use App\Models\User;

    $user = User::factory()->create();

    $user->delete();

    $this->assertModelMissing($user);

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

`expectsDatabaseQueryCount`メソッドは、テスト中に実行されるであろうデータベースクエリの総数をアサートします。実際に実行されたクエリの数がこの期待値と一致しない場合、テストは失敗します。

    $this->expectsDatabaseQueryCount(5);

    // テスト…
