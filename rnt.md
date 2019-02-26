No bundle URL present
lsof -i:8081
kill -9 63701 

React-Navigation undefined is not an object (evaluating 'RNGestureHandlerModule.State'）

react-navigation 更新了3.0版本

yarn add react-native-gesture-handler -S

react-native link react-native-gesture-handler

```js
import React from "react";
import { View, Text } from "react-native";
import { createStackNavigator, createAppContainer } from "react-navigation";

class HomeScreen extends React.Component {
  render() {
    return (
      <View style={{ flex: 1, alignItems: "center", justifyContent: "center" }}>
        <Text>Home Screen</Text>
      </View>
    );
  }
}

const AppNavigator = createStackNavigator({
  Home: {
    screen: HomeScreen
  }
});

export default createAppContainer(AppNavigator);


```

VirtualizedList: missing keys for items, make sure to specify a key property on each item or provide
https://reactnative.cn/docs/0.51/flatlist.html#content

