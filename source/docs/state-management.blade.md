---
title: State Management
extends: _layouts.documentation
section: content
---

Livewire components store and track state using public class properties on the Component class.

@code(['lang' => 'php'])
@verbatim
class FooComponent extends Component
{
    // Public Property
    public $foo;
@endverbatim
@endcode

Here are some helpful points about public properties in Livewire.

@table
Public Properties |
--- |
Automatically available via the `$this` object inside the component's Blade view. |
Can be used for data-binding (`public $foo;` can be bound via `wire:model="foo"`). |
Are sent back and forth with every network request (increase network payload). |
Should not store sensitive data. (any information stored in them will be visible to JavaScript). |
They MUST be of PHP type: `null`, `string`, `numeric`, `boolean`, or `array` (because JavaScript has to be able to understand them) |
@endtable

@warning
<code>protected</code> and <code>private</code> properties DO NOT persist between Livewire updates. In general, you should avoid using them for storing state.
@endwarning

### Casting Properties {#casting-properties}

Livewire offers an api to "cast" public properties to a more usable data type. Two common use-cases for this are working with date objects like `Carbon` instances, and dealing with Laravel collections:

@code(['lang' => 'php'])
@verbatim
class CastedComponent extends Component
{
    public $options = ['foo', 'bar', 'bar'];
    public $expiresAt = 'tomorrow';

    protected $casts = [
        'options' => 'collection',
        'expiresAt' => 'date',
    ];

    public function getUniqueOptions()
    {
        return $this->options->unique();
    }

    public function getExpirationDateForHumans()
    {
        return $this->expiresAt->format('m/d/Y');
    }

    ...
@endverbatim
@endcode

#### Custom Casters {#custom-casters}

Livewire allows you to build your own custom casters for custom use-cases. Implement the `Livewire\Castable` interface in a class and reference it in the `$casts` property:

@code(['lang' => 'php'])
@verbatim
class FooComponent extends Component
{
    public $foo = ['bar', 'baz'];

    protected $casts = ['foo' => CollectionCaster::class];

    ...
@endverbatim
@endcode

@code(['lang' => 'php'])
@verbatim
class CollectionCaster implements Castable
{
    public function cast($value)
    {
        return collect($value);
    }

    public function uncast($value)
    {
        return $value->all();
    }
}
@endverbatim
@endcode


### Initializing Properties {#initializing-properties}

Let's say you wanted to make the 'Hello World' message more specific, and greet the currently logged in user. You might try setting the message property to:

@code(['lang' => 'php'])
public $message = 'Hello ' . auth()->user()->first_name;
@endcode

Unfortunately, this is invalid PHP (you can't assign the result of an expression to a property directly). However, you can initialize properties using the `mount` method/hook in Livewire. For example:

@codeComponent([
    'className' => 'HelloWorld.php',
    'viewName' => 'hello-world.blade.php',
])
@slot('class')
use Livewire\Component;

class HelloWorld extends Component
{
    public $message;

    public function mount()
    {
        $this->message = 'Hello ' . auth()->user()->first_name;
    }

    public function render()
    {
        return view('livewire.hello-world');
    }
}
@endslot
@slot('view')
@verbatim
<div>
    <h1>{{ $this->message }}</h1>
</div>
@endverbatim
@endslot
@endcodeComponent

You can think of `mount()` like you would the `boot()` method of a Laravel Model, or the `created()` method of a Vue component.

### Storing/Referencing Eloquent Models {#storing-eloquent-models}

It is common to want to set an Eloquent model (like `App\User`) as a public property (`public $user`), HOWEVER, this is not allowed. Public properties can only be set as non-object values (arrays, integers, strings, booleans).

The idiomatic way to store/retrieve Eloquent model's inside a Livewire component, is to use computed properties. [Learn more about them here](/docs/computed-properties).

@codeComponent([
    'className' => 'ShowUser.php',
    'viewName' => 'show-user.blade.php',
])
@slot('class')
use Livewire\Component;

class ShowUser extends Component
{
    public $userId;

    public function mount($userId)
    {
        $this->userId = $userId;
    }

    public function getUserProperty()
    {
        return \App\User::find($this->userId);
    }

    public function executeSomeActionOnTheUser()
    {
        $this->user->someAction();
    }

    public function render()
    {
        return view('livewire.show-user');
    }
}
@endslot
@slot('view')
@verbatim
<div>
    <h1>Hi {{ $this->user->name }}!</h1>

    <button type="button" wire:click="executeSomeActionOnTheUser">Do Something</button>
</div>
@endverbatim
@endslot
@endcodeComponent
