# react-native-sweet-record

An ActiveRecord style layer that sits on top of React Native's AsyncStorage.

### Introduction

Let me begin by saying that you'd have to be as bat shit crazy as a Donald Trump voter walking through Harlem with one of those bright red, obnoxious, "Make America Great Again" hats to use react-native-sweet-record for any sort of large-scale persistent data storage project. It is intended to be used for hundreds of records, not thousands. If your requirements are of that magnitude please investigate other react native solutions with an actual database behind them. 

However, if you're looking for a bad-ass, mother-loving, salt-of-the-earth, ActiveRecord style library that doesn't require anything more than what the beautiful people behind the React Native AsyncStorage component have provided us then look no further. React-native-sweet-record is your huckleberry. 

You have concerns. I get it. That is not exactly a ringing endorsement. Not many project README's start out telling you that you're insane if you use them. Well, I can tell you that react-native-sweet-record has been used for all persistent data storage on [**"The Comedy Companion"**](https://github.com/dereksweet/ComedyCompanion), an app that is used by hundreds upon hundreds of stand-up comedians around the world, who all have hundreds of jokes, and not one of them has complained to me about the speed or reliability of the library. So, I decided to release it as a stand-alone project. I like it, I find it easy to use, and I'm hoping others do too!

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
    return "my_table_name";
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

It may look like there is bit of heavy use of the `defaults` object there, but I assure you there is a method to the madness. That may be a bit of a hyperbole to use the word "madness" in such a context but just stick with me. 

Now that you've defined your model you are free to include it in any of your other object files. Simply add the following import statement at the top of your file: 

```
import MyModel from './models/MyModel';
```

... and then you get all sorts of powerful data storage and retrieval for the model you have defined. For example, to create a new instance of your model all you need to do is type `let my_model_1 = new MyModel()` and you've got your object. You can now set any of the attributes for your model by simply typing `my_model_1._field1_integer = 3`, or `my_model_1._field2_string = 'blah'`, or `my_model_1._field3_array = ['this','is','awesome']` and the values will be associated with the fields. When you're ready to save your instance to AsyncStorage simply type `my_model_1.save()` and it's done. React-native-sweet-record will take care of assigning your instance an `_id` and persisting it to the device storage for later retrieval. Getting your object back from AsyncStorage is as simple as typing `MyModel.get(1).then((my_model) => this.my_model_1_copy = my_model)` (presuming your instance `_id` is 1), to which modifications can be made to the attributes, and then saved again to overwrite the previous version.

See immediately below this for a list of all the available commands. I think you'll agree that they are scrumptious. 

### Available Commands

#### .get(id, refresh_models=false)

To retrieve an instance from AsyncStorage simply type something like the following. This would be if you wanted to retrieve the instance with an `_id` of 1:

```
MyModel.get(1).then((my_model) => this.my_model_1_copy = my_model);
```

The `refresh_models` parameter tells react-native-sweet-record that you want to refresh the objects for any attributes that happen to be nested SweetModel objects. If you set it to true, then it will use the `_id` of the nested SweetModel object to pull it's instance from AsyncStorage, then update and save the parent object instance.

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

The `refresh_models` parameter tells react-native-sweet-record that you want to refresh the objects for any attributes that happen to be nested SweetModel objects within any of the results returned by `all()`. If you set it to true, then it will use the `_id` of the nested SweetModel objects to pull their instances from AsyncStorage, then update and save the parent object instanecs.

### Examples

To see some production level usage of the library, please see the [**src/models/**](https://github.com/dereksweet/ComedyCompanion/tree/master/src/models) folder for "The Comedy Companion" project. There are 4 models within that exhibit, between them, all of the major features of react-native-sweet-record. You'll see examples of how to properly use dates, arrays, hashes, and even other models, as field types. If it can be serialized in JSON, then react-native-sweet-record can manage that shit. 
