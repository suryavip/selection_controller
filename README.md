# Selector
Tools to help manage selection state.

## Example use case
You have `User` class like this:
```dart
class User {
	String id;
	String name;
	int age;

	String get ageGroup {
		if (age >= 0 && age <= 12) return 'child';
		if (age >= 12 && age <= 18) return 'teen';
		return 'adult';
	}

	User({
		required this.id;
		required this.name;
		required this.age;
	});
}
```

And you have some `User`s as an option:
```dart
List<User> users = [
	User(id: 'u0', name: 'James', age: 24), // adult
	User(id: 'u1', name: 'Mike', age: 17), // teen
	User(id: 'u2', name: 'John', age: 16), // teen
	User(id: 'u3', name: 'Harry', age: 28), // adult
	User(id: 'u4', name: 'Kid', age: 12), // child
];
```

Then you can use `Selector` to contain and manage the selection state.
```dart
Selector<User> userSelector = Selector(
	choosableOptions: users,
	indexGetter: (user) => user.id, // options will be indexed by the user's id
);
```

You can use provided methods to check and manipulate a user selected status:
```dart
userSelector.isSelected('u0'); // return true if James selected (check by index which is the user's id)
userSelector.isSelected(users[0]); // return true if James selected (check by the actual User object)

userSelector.select('u0'); // select James
userSelector.select(users[0]); // also select James

userSelector.unselect('u1'); // unselect Mike
userSelector.unselect(users[1]); // also unselect Mike

userSelector.toggle('u2'); // toggle John
userSelector.toggle(users[2]); // also toggle John

userSelector.selected; // to get all Users that currently selected.
```

You can check and manipulate multiple users too:
```dart
userSelector.isAllSelected; // return true if all users selected
userSelector.isNoneSelected; // return true if no user selected

userSelector.selectAll(); // select all users
userSelector.unselectAll(); // unselect all users

userSelector.toggleAll(); // unselect all users if all users are selected. Else, select all users.
```

## Constraint
If you only allow 1 user to be selected at a time:
```dart
Selector<User> userSelector = Selector(
	choosableOptions: users,
	indexGetter: (user) => user.id,
	mode: SelectorMode.single, // apply constraint that only allow 1 option selected at a time.
);

userSelector.select('u0'); // select James

userSelector.isSelected('u0'); // James: true
userSelector.isSelected('u1'); // Mike: false
userSelector.isSelected('u2'); // John: false
...

userSelector.select('u1'); // select Mike. James automatically deselected.

userSelector.isSelected('u0'); // James: false
userSelector.isSelected('u1'); // Mike: true
userSelector.isSelected('u2'); // John: false
...
```

If you need some kind of grouping and only allow selection on one group:
```dart
Selector<User> userSelector = Selector(
	choosableOptions: users,
	indexGetter: (user) => user.id,
	groupNameGetter: (user) => user.ageGroup, // to generate group name from each options (users)
	mode: SelectorMode.sameGroup, // apply constraint that only allow selected options to be on a same group (same age group in this case).
);

userSelector.select('u0'); // select James

userSelector.isSelected('u0'); // James: true
userSelector.isSelected('u1'); // Mike: false
userSelector.isSelected('u2'); // John: false
userSelector.isSelected('u3'); // Harry: false
...
userSelector.isGroupSelected('adult'); // true
userSelector.isGroupSelected('teen'); // false
userSelector.isGroupSelectedAll('adult'); // false
userSelector.isGroupSelectedAll('teen'); // false

userSelector.select('u3'); // select Harry. James still selected because Harry and James are on the same age group (adult).

userSelector.isSelected('u0'); // James: true
userSelector.isSelected('u1'); // Mike: false
userSelector.isSelected('u2'); // John: false
userSelector.isSelected('u3'); // Harry: true
...
userSelector.isGroupSelected('adult'); // true
userSelector.isGroupSelected('teen'); // false
userSelector.isGroupSelectedAll('adult'); // true
userSelector.isGroupSelectedAll('teen'); // false

userSelector.select('u2'); // select John. James and Harry will automatically deselected since John is on teen age group.

userSelector.isSelected('u0'); // James: false
userSelector.isSelected('u1'); // Mike: false
userSelector.isSelected('u2'); // John: true
userSelector.isSelected('u3'); // Harry: false
...
userSelector.isGroupSelected('adult'); // false
userSelector.isGroupSelected('teen'); // true
userSelector.isGroupSelectedAll('adult'); // false
userSelector.isGroupSelectedAll('teen'); // false

userSelector.select('u1'); // select Mike. John still selected because Mike and John are on the same age group (teen).

userSelector.isSelected('u0'); // James: false
userSelector.isSelected('u1'); // Mike: true
userSelector.isSelected('u2'); // John: true
userSelector.isSelected('u3'); // Harry: false
...
userSelector.isGroupSelected('adult'); // false
userSelector.isGroupSelected('teen'); // true
userSelector.isGroupSelectedAll('adult'); // false
userSelector.isGroupSelectedAll('teen'); // true
```

Select and unselect multiple group:
```dart
userSelector.selectGroup('adult'); // select all adult.

userSelector.isGroupSelected('adult'); // true
userSelector.isGroupSelectedAll('adult'); // true

userSelector.unselectGroup('adult'); // unselect all adult.

userSelector.isGroupSelected('adult'); // false
userSelector.isGroupSelectedAll('adult'); // false
```

Additional methods: see documentation.