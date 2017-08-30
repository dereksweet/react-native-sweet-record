# react-native-sweet-record

An ActiveRecord style layer that sits on top of React Native's AsyncStorage.

### Introduction

Let me begin by saying that you'd have to be as bat shit crazy as a Donald Trump voter walking through Harlem with one of those bright red, obnoxious, "Make America Great Again" hats to use react-native-sweet-record for any sort of large-scale persistent data storage project. It is intended to be used for hundreds of records, not thousands. If your requirements are of that magnitude please investigate other react native solutions with an actual database behind them. 

However, if you're looking for a bad-ass, mother-loving, salt-of-the-earth, ActiveRecord style library that doesn't require anything more than what the beautiful people behind the React Native AsyncStorage component have provided us then look no further. React-native-sweet-record is your huckleberry. 

You have concerns. I get it. That is not exactly a ringing endorsement. Not many project README's start out telling you that you're insane if you use them. Well, I can tell you that react-native-sweet-record has been used for all persistent data storage on [**"The Comedy Companion"**](https://github.com/dereksweet/ComedyCompanion), an app that is used by hundreds of stand-up comedians around the world, who all have hundreds of jokes, and not one of them has complained to me about the speed or reliability of the library. So, I decided to release it as a stand-alone project. I like it, I find it easy to use, and I'm hoping others do too!

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

### Examples

To see some production level usage of the library, please see the [**src/models/**](https://github.com/dereksweet/ComedyCompanion/tree/master/src/models) folder for "The Comedy Companion" project. There are 4 models within that exhibit, between them, all of the major features of react-native-sweet-record. You'll see examples of how to properly use dates, arrays, hashes, and even other models, as field types. If it can be serialized in JSON, then react-native-sweet-record can manage that shit. 
