# expense-tracker
App.js
```
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { ExpenseProvider } from './ExpenseContext';
import { HomeScreen, AddExpenseScreen, ChartsScreen } from './screens';
import LandingPage from './LandingPage';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

const HomeTabs = () => (
  <Tab.Navigator>
    <Tab.Screen name="Home" component={HomeScreen} />
    <Tab.Screen name="Charts" component={ChartsScreen} />
  </Tab.Navigator>
);

const App = () => {
  return (
    <ExpenseProvider>
      <NavigationContainer>
        <Stack.Navigator initialRouteName="LandingPage">
          <Stack.Screen name="LandingPage" component={LandingPage} options={{ headerShown: false }} />
          <Stack.Screen name="HomeTabs" component={HomeTabs} options={{ headerShown: false }} />
          <Stack.Screen name="AddExpense" component={AddExpenseScreen} />
        </Stack.Navigator>
      </NavigationContainer>
    </ExpenseProvider>
  );
};

export default App;
```

Bottom Tab Navigator
```
// BottomTabNavigator.js
import React from 'react';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import HomeScreen from './screens/HomeScreen';
import ChartsScreen from './screens/ChartsScreen';

const Tab = createBottomTabNavigator();

const BottomTabNavigator = () => {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Charts" component={ChartsScreen} />
    </Tab.Navigator>
  );
};

export default BottomTabNavigator;
```
ExpenseContext.js 
```
// ExpenseContext.js
import React, { createContext, useState } from 'react';

const ExpenseContext = createContext();

export const ExpenseProvider = ({ children }) => {
  const [expenses, setExpenses] = useState([]);

  const addExpense = (expense) => {
    setExpenses([...expenses, expense]);
  };

  return (
    <ExpenseContext.Provider value={{ expenses, addExpense }}>
      {children}
    </ExpenseContext.Provider>
  );
};

export default ExpenseContext;
```
Landing Page .js
```
// LandingPage.js
import React from 'react';
import { View, Text, StyleSheet, Button } from 'react-native';

const LandingPage = ({ navigation }) => {
    const handleGetStarted = () => {
      // Navigate to the correct screen name 'HomeTabs'
      navigation.navigate('HomeTabs');
    };
  
    return (
      <View style={styles.container}>
        <Text style={styles.title}>Welcome to Expense Tracker</Text>
        <Text style={styles.subtitle}>Track your expenses and manage your finances easily!</Text>
        <Button title="Get Started" onPress={handleGetStarted} />
      </View>
    );
  };

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    textAlign: 'center',
  },
  subtitle: {
    fontSize: 16,
    marginBottom: 40,
    textAlign: 'center',
  },
});

export default LandingPage;
```
Add expense screen.js
```
// AddExpenseScreen.js
import React, { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';

const AddExpenseScreen = ({ navigation }) => {
  const [amount, setAmount] = useState('');
  const [description, setDescription] = useState('');
  const [date, setDate] = useState('');

  const handleAddExpense = () => {
    // Validate input fields
    if (!amount || !description || !date) {
      alert('Please fill in all fields.');
      return;
    }

    // Save expense (you can implement saving logic here)

    // Navigate back to the home screen
    navigation.goBack();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Add Expense</Text>
      <TextInput
        style={styles.input}
        placeholder="Amount"
        value={amount}
        onChangeText={setAmount}
        keyboardType="numeric"
      />
      <TextInput
        style={styles.input}
        placeholder="Description"
        value={description}
        onChangeText={setDescription}
      />
      <TextInput
        style={styles.input}
        placeholder="Date (YYYY-MM-DD)"
        value={date}
        onChangeText={setDate}
      />
      <Button title="Save Expense" onPress={handleAddExpense} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
  },
  input: {
    height: 40,
    borderColor: 'gray',
    borderWidth: 1,
    marginBottom: 20,
    paddingHorizontal: 10,
  },
});

export default AddExpenseScreen;
```
Charts Screen.js
```
import React, { useContext } from 'react';
import { View, StyleSheet, FlatList, Text } from 'react-native';
import { PieChart } from 'react-native-chart-kit';
import ExpenseContext from '../ExpenseContext';

const ChartsScreen = () => {
  const { expenses } = useContext(ExpenseContext);

  // Group expenses by description and calculate total amount for each description
  const expensesByDescription = expenses.reduce((acc, expense) => {
    acc[expense.description] = acc[expense.description] ? acc[expense.description] + parseFloat(expense.amount) : parseFloat(expense.amount);
    return acc;
  }, {});

  // Calculate total expenses
  const totalExpenses = expenses.reduce((acc, expense) => acc + parseFloat(expense.amount), 0);

  // Format data for PieChart and generate colors
  let colors = ['#FF5733', '#FFBD33', '#33FFA8', '#338AFF', '#9B33FF', '#FF33E3', '#A833FF', '#FF3362', '#33FFE9', '#33FF33'];
  const chartData = Object.keys(expensesByDescription).map((description, index) => ({
    name: description,
    amount: expensesByDescription[description],
    color: colors[index % colors.length]
  }));

  // Function to render each expense item in the table
  const renderItem = ({ item }) => (
    <View style={styles.item}>
      <Text style={styles.description}>{item.name}</Text>
      <Text style={styles.amount}>{item.amount}</Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <PieChart
        data={chartData}
        width={300}
        height={200}
        chartConfig={{
          backgroundColor: '#ffffff',
          backgroundGradientFrom: '#ffffff',
          backgroundGradientTo: '#ffffff',
          color: (opacity = 1) => `rgba(0, 0, 0, ${opacity})`,
        }}
        accessor="amount"
        backgroundColor="transparent"
        paddingLeft="15"
        absolute
      />
      <Text style={styles.total}>Total Expenses: ${totalExpenses.toFixed(2)}</Text>
      <Text style={styles.tableTitle}>Expenses:</Text>
      <FlatList
        data={expenses} // Use original expenses data
        renderItem={renderItem}
        keyExtractor={(item, index) => index.toString()} // Use index as key
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f0f0f0',
    padding: 20,
  },
  total: {
    fontSize: 20,
    fontWeight: 'bold',
    marginTop: 10,
  },
  tableTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginTop: 20,
    marginBottom: 10,
  },
  item: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    borderBottomWidth: 1,
    borderBottomColor: '#ccc',
    paddingVertical: 10,
  },
  description: {
    flex: 1,
    marginRight: 10,
  },
  amount: {
    width: 80,
    textAlign: 'right',
  },
});

export default ChartsScreen;
```
Homescreen.js
```
// HomeScreen.js
import React, { useContext, useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import ExpenseContext from '../ExpenseContext';

const HomeScreen = () => {
  const { addExpense } = useContext(ExpenseContext);
  const [amount, setAmount] = useState('');
  const [description, setDescription] = useState('');

  const handleAddExpense = () => {
    if (!amount || !description) return;
    const newExpense = {
      amount,
      description,
      date: new Date().toLocaleString()
    };
    addExpense(newExpense);
    setAmount('');
    setDescription('');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Add New Expense</Text>
      <TextInput
        style={styles.input}
        placeholder="Amount"
        value={amount}
        onChangeText={setAmount}
        keyboardType="numeric"
      />
      <TextInput
        style={styles.input}
        placeholder="Description"
        value={description}
        onChangeText={setDescription}
      />
      <Button title="Add Expense" onPress={handleAddExpense} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f0f0f0',
    padding: 20,
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
  },
  input: {
    width: '100%',
    marginBottom: 20,
    padding: 10,
    borderWidth: 1,
    borderColor: '#ccc',
    borderRadius: 5,
    backgroundColor: '#fff',
  },
});

export default HomeScreen;
```
index.js
```
// screens/index.js

export { default as HomeScreen } from './HomeScreen';
export { default as AddExpenseScreen } from './AddExpenseScreen';
export { default as ChartsScreen } from './ChartsScreen';
// Export additional screen components as needed
  ```
Landing Page Screenshot
![Landing Page Screenshot]
