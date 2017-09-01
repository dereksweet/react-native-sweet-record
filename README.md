# react-native-sweet-record

An ActiveRecord style layer that sits on top of React Native's AsyncStorage.

### Introduction

Let me begin by saying that you'd have to be bat shit crazy to use react-native-sweet-record for any sort of large-scale persistent data storage project. It is intended to be used for hundreds of records, not thousands. If your requirements are of that magnitude please investigate other react native solutions with an actual database behind them. 

However, if you're looking for a bad-ass, mother-loving, salt-of-the-earth, ActiveRecord style library that doesn't require anything more than what the beautiful people behind the React Native AsyncStorage component have provided us then look no further. React-native-sweet-record is your huckleberry. 

You have concerns. I get it. That is not exactly a ringing endorsement. Not many project README's start out telling you that you're insane if you use them. Well, I can tell you that react-native-sweet-record has been used for all persistent data storage on [**"The Comedy Companion"**](https://github.com/dereksweet/ComedyCompanion), an app that is used by hundreds upon hundreds of stand-up comedians around the world, who all have hundreds upon hundreds of jokes, and not one of them has complained to me about the speed or reliability of the library. So, I decided to release it as a stand-alone project. I like it, I find it easy to use, and I'm hoping others do too!

### Installation

From a terminal navigate to your project folder and type: 

`npm install react-native-sweet-record --save`

or add the following line to your dependencies within package.json and then `npm install`

`"react-native-sweet-record": "1.0.0"`

### Usage

In order to use react-native-sweet-record you are going to probably want to create a folder in your project to keep all your model definitions. Call me crazy, but I like to call mine `models`. I'm just a rebel that way I guess.

Within your model folder have one file per model that has the following basic structure:

```
import SweetModel from 'react-native-sweet-model';

const defaults = {
  _field1_integer: 0,
  _field2_string: '',
  _field3_array: []
};

export default class MyModel extends SweetModel {
  static databaseName() {
    return "MyDatabaseName";
  }

  static tableName() {
    return "my_models";
  }

  static dateFields() {
    return ['_created_at', '_updated_at'];
  }

  constructor(data = {
    _id: -1,
    _field1_integer: defaults._field1_integer,
    _field2_string: defaults._field2_string,
    _field3_array: defaults._field3_array,
    _created_at: new Date(),
    _updated_at: new Date()
  })
  {
    super();

    this._id             = data._id;
    this._field1_integer = data._field1_integer || defaults._field1_integer;
    this._field2_string  = data._field2_string || defaults._field2_string;
    this._field3_array   = data._field3_array || defaults._field3_array;
    this._created_at     = data._created_at;
    this._updated_at     = data._updated_at;
  }
}
```

It may look like there is bit of heavy use of the `defaults` object there, but I assure you there is a method to the madness. Following this pattern allows you to automatically migrate old records as new attributes are added over time. Eventually I'd like to see this all abstracted into some kind of easily digested .yml file or something.

There are 4 static functions that can be overridden, two which must be overridden and two that are optional. 

The two required methods that must exist in your model are `databaseName()` and `tableName()`. These are necessary in defining the prefix that will be prepended to the key used to reference every instance of your model that you store. They should both simply return a string, I recommend a CamelCase string for `databaseName()` and a snake_case string for `tableName()`. Every model should share the same `databaseName()` and have a unique `tableName()`. 

Think of these as roughly equivalent to the database instance name and table names found for a traditional model representation in a relational database. As mentioned the strings will be used to simply build a unique key for each object in the AsyncStorage key/value storage system. For example, using the basic model above the first object created of type MyModel would get saved under the key `@MyDatabaseName:my_models/1`, not that you will ever need to know that level of detail to use the library, but am trying to provide a little insight into how this library works. 

The two optional methods that must exist if you want certain fields to behave appropriately are `dateFields()` and `modelFields()`. These must return arrays defining which attributes you have that are of type DateTime or a nested SweetModel. Since the attribute values must be serialized into a JSON string and read back out it's imperative that the system know how to properly encode them upon retrieval. Hopefully from the example above it is clear how to use `dateFields()`. An example of `modelFields()` will be provided in the "Nested Models" section below. 

Now that you've defined your model you are free to include it in any of your other object files. Simply add the following import statement at the top of your file: 

```
import MyModel from './models/MyModel';
```

... and then you get all sorts of powerful data storage and retrieval for the model you have defined. For example, to create a new instance of your model all you need to do is type:

`let my_model_1 = new MyModel()`

... and you've got your object. You can now set any of the attributes for your model by simply typing things like:

```
my_model_1._field1_integer = 3
my_model_1._field2_string = 'blah'
my_model_1._field3_array = ['this','is','awesome']
```

...and the values will be associated with the fields. When you're ready to save your instance to AsyncStorage simply type:

```
my_model_1.save()
```

... and it's done. React-native-sweet-record will take care of assigning your instance an `_id` and persisting it to the device storage for later retrieval. The instance you just saved will have it's `_id` set as well. Getting your object back from AsyncStorage is as simple as typing, presuming your instance `_id` is 1:

```
MyModel.get(1).then((my_model) => this.my_model_1_copy = my_model)
```
... to which modifications can be made to the attributes, and then saved again to overwrite the previous version.

See immediately below this for a list of all the available commands. I think you'll agree that they are scrumptious. 

### Available Commands

#### .get(id, refresh_models=false)

To retrieve an instance from AsyncStorage simply type something like the following. This would be if you wanted to retrieve the instance with an `_id` of 1:

```
MyModel.get(1).then((my_model) => this.my_model_1_copy = my_model);
```

The `refresh_models` parameter tells react-native-sweet-record that you want to refresh the objects for any attributes that happen to be nested SweetModel objects. If you set it to true, then it will use the `_id` of the nested SweetModel object to pull it's instance from AsyncStorage, then update and save the parent object instance. If you want your nested objects to remain consistent with their sibling objects then set `refresh_models` to true. If, however, you want your nested object to be more of a snapshot of what it was when the parent object was saved then keep it false. 

#### .save(callback = null)

To save an instance of your model to AsyncStorage, simply type something like the following: 

```
my_model_1.save();
```

You have the option of providing a callback function to be run once the save operation is complete. This is so you can achieve some form of synchronous operation in case you have UI elements that need to be updated when the object has been completely persisted to storage, or any other such need for synchronous behavior. 

#### .destroy(callback = null)

To permanently remove an instance of your model from AsyncStorage, simply type something like the following:

```
my_model_1.destroy();
```

You have the option of providing a callback function to be run once the destroy operation is complete. This is so you can achieve some form of synchronous operation in case you have UI elements that need to be updated when the object has been completely removed from storage, or any other such need for synchronous behavior. 

#### .all(sort_field = '_id', sort_order = 'ASC', refresh_models = false)

To retrieve all instances of your model from AsyncStorage, returned as an array of objects, simply type something like the following: 

```
MyModel.all().then((my_models) => this.all_my_models = my_models);
```

`all_my_models` will then contain an array of objects that can be treated just like any other instance of your object. For example change the value of an attribute for the 2nd model in your results simply type `this.all_my_models[1]._field1_integer = 5` then `this.all_my_models[1].save()` and that 2nd object should be updated in memory AND in the persisted storage.

You have the option of passing in a `sort_field` and a `sort_order` for ordering your results. Simply pass in the name of the attribute, as a string, to `sort_field` and either `ASC` or `DESC` to `sort_order` for ascending and descending orders respectively. 

The `refresh_models` parameter tells react-native-sweet-record that you want to refresh the objects for any attributes that happen to be nested SweetModel objects within any of the results returned by `all()`. If you set it to true, then it will use the `_id` of the nested SweetModel objects to pull their instances from AsyncStorage, then update and save the parent object instances. If you want your nested objects to remain consistent with their sibling objects then set `refresh_models` to true. If, however, you want your nested object to be more of a snapshot of what it was when the parent object was saved, then keep it false. 

#### .destroy_all(callback = null)

To destroy all instances of your model from AsyncStorage simply type something like the following:

```
MyModel.destroy_all()
```

This will completely wipe every instance of your model from AsyncStorage so be careful when using it! You have the option of providing a callback function to be run once the destroy_all operation is complete. This is so you can achieve some form of synchronous operation in case you have UI elements that need to be updated when the objects have been completely removed from storage, or any other such need for synchronous behavior.  

#### .where(filter_hash, operation = 'AND', sort_field = '_id', sort_order= 'ASC', refresh_models = false)

To retrieve a filtered array of your models based on your specifications this is how you do it. Let me start by saying this can be extremely slow, so use it wisely. AsyncStorage is just a key/value storage system so the only way that react-native-sweet-record can filter results is by pulling every single instance of your model, looping through them, and plucking out the ones that match your criteria. If you ignore my recomendation at the start of this README and use this library to manage thousands upon thousands of records the `where()` method is where you are going to feel the pain of a million dickslaps.

The only required parameter to `where()` is the `filter_hash`. What you want to want to pass in for this parameter is a hash that represents what would be your `WHERE` clauses in a relational database. The key for each member of the hash should be the name of the attribute you want to filter on, and the value should be a string that contains two parts separated by a vertical bar character. The first half of the value string is the operator you want to use in the comparison, and the second half is the value you want to compare it to. 

For example, if I wanted to get all instances of the MyModel, defined above, that had a value for `field1_integer` that was greater than 3 and had a value for `field2_string` that contained the word "fartknuckle" I would type something like the following:

```
MyModel.where({"field1_integer":"GT|3", "field2_string":"LIKE|'fartknuckle'"})
  .then((my_filtered_models) => this.my_filtered_models = my_filtered_models);
```

The available operators to use in the first half of a value string for `filter_hash` are: `EQ`, `GT`, `GTE`, `LT`, `LTE`, and `LIKE`. Hopefully they are all self-explanatory. For the nooblets that are reading this `LTE` and `GTE` stand for "Less-Than-Or-Equal-To" and "Greater-Than-Or-Equal-To" respectively. 

The remaining optional parameters to the `where()` method are `operation`, `sort_field`, `sort_order`, and `refresh_models`.

`operation` can be either `AND` or `OR`. This is the conjunction that is placed between the where clauses defined by your `filter_hash`. If you were to pass in `OR` for the example just above, that would give you any instance of MyModel that had a value for `field1_integer` that was greater than 3 OR it had a value for `field2_string` that contained the word "fartknuckle". Currently there is no way to combine `AND` and `OR` conjunctions, it's one or the other.

You have the option of passing in a `sort_field` and a `sort_order` for ordering your results. Simply pass in the name of the attribute, as a string, to `sort_field` and either `ASC` or `DESC` to `sort_order` for ascending and descending orders respectively. 

The `refresh_models` parameter tells react-native-sweet-record that you want to refresh the objects for any attributes that happen to be nested SweetModel objects within any of the results returned by `where()`. If you set it to true, then it will use the `_id` of the nested SweetModel objects to pull their instances from AsyncStorage, then update and save the parent object instances. If you want your nested objects to remain consistent with their sibling objects then set `refresh_models` to true. If, however, you want your nested object to be more of a snapshot of what it was when the parent object was saved, then keep it false. 

### Nested Models

As you've for sure read above by now, it is possible to nest SweetModel object as attributes of other SweetModel objects! Since each SweetModel can be serialized as JSON, and every attribute in a SweetModel gets seralized into JSON, it's entirely possible to nest one SweetModel within another. You just need to let react-native-sweet-model know which attributes are nested models so when they are retrieved it knows to intialize them as such.

For example, let's say we wanted to add a new nested model, MyOtherModel, to MyModel defined at the beginning of this readme. The file contents for your model definition would now look something like this: 

```
import SweetModel from 'react-native-sweet-model';
import MyOtherModel from './MyOtherModel`

const defaults = {
  _field1_integer: 0,
  _field2_string: '',
  _field3_array: [],
  _field4_model: new MyOtherModel()
};

export default class MyModel extends SweetModel {
  static databaseName() {
    return "MyDatabaseName";
  }

  static tableName() {
    return "my_models";
  }

  static dateFields() {
    return ['_created_at', '_updated_at'];
  }
  
  static modelFields() {
    return [{ field: '_field4_model', class: MyOtherModel, array: false}];
  }

  constructor(data = {
    _id: -1,
    _field1_integer: defaults._field1_integer,
    _field2_string: defaults._field2_string,
    _field3_array: defaults._field3_array,
    _field4_model: defaults._field4_model,
    _created_at: new Date(),
    _updated_at: new Date()
  })
  {
    super();

    this._id             = data._id;
    this._field1_integer = data._field1_integer || defaults._field1_integer;
    this._field2_string  = data._field2_string || defaults._field2_string;
    this._field3_array   = data._field3_array || defaults._field3_array;
    this._field4_model   = new MyOtherModel(data._field4_model || defaults._field4_model);
    this._created_at     = data._created_at;
    this._updated_at     = data._updated_at;
  }
}
```

Each element of `modelFields()` must be a hash with 3 keys and corresponding values. `field` specifies that name of the attribute that is a nested model, `class` specifies what type of nested model it is, and `array` should be either true or false to tell the system if it is just a single instance of the nested model or an array of them. To see an example of a nested model that is an array see the [**SetList**](https://github.com/dereksweet/ComedyCompanion/blob/master/src/models/set_list.js) model for "The Comedy Companion". 

### Examples

To see some production level usage of the library, please see the [**src/models/**](https://github.com/dereksweet/ComedyCompanion/tree/master/src/models) folder for "The Comedy Companion" project. There are 4 models within that exhibit, between them, all of the major features of react-native-sweet-record. You'll see examples of how to properly use dates, arrays, hashes, and even other nested Sweetmodels, as attributes. If it can be serialized in JSON, then react-native-sweet-record can manage it. 
