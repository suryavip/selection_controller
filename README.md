# SelectionController
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

Then you can use `SelectionController` to contain and manage the selection state.
```dart
SelectionController<User> selection = SelectionController(
	choosableOptions: users,
	indexGetter: (user) => user.id, // options will be indexed by the user's id
);
```

You can use provided methods to check and manipulate a user selected status:
```dart
selection.isSelected('u0'); // return true if James selected (check by index which is the user's id)
selection.isSelected(users[0]); // return true if James selected (check by the actual User object)

selection.select('u0'); // select James
selection.select(users[0]); // also select James

selection.unselect('u1'); // unselect Mike
selection.unselect(users[1]); // also unselect Mike

selection.toggle('u2'); // toggle John
selection.toggle(users[2]); // also toggle John

selection.selected; // to get all Users that currently selected.
```

You can check and manipulate multiple users too:
```dart
selection.isAllSelected; // return true if all users selected
selection.isNoneSelected; // return true if no user selected

selection.selectAll(); // select all users
selection.unselectAll(); // unselect all users

selection.toggleAll(); // unselect all users if all users are selected. Else, select all users.
```

## Constraint
If you only allow 1 user to be selected at a time:
```dart
SelectionController<User> selection = SelectionController(
	choosableOptions: users,
	indexGetter: (user) => user.id,
	mode: SelectorMode.single, // apply constraint that only allow 1 option selected at a time.
);

selection.select('u0'); // select James

selection.isSelected('u0'); // James: true
selection.isSelected('u1'); // Mike: false
selection.isSelected('u2'); // John: false
...

selection.select('u1'); // select Mike. James automatically deselected.

selection.isSelected('u0'); // James: false
selection.isSelected('u1'); // Mike: true
selection.isSelected('u2'); // John: false
...
```

If you need some kind of grouping and only allow selection on one group:
```dart
SelectionController<User> selection = SelectionController(
	choosableOptions: users,
	indexGetter: (user) => user.id,
	groupNameGetter: (user) => user.ageGroup, // to generate group name from each options (users)
	mode: SelectorMode.sameGroup, // apply constraint that only allow selected options to be on a same group (same age group in this case).
);

selection.select('u0'); // select James

selection.isSelected('u0'); // James: true
selection.isSelected('u1'); // Mike: false
selection.isSelected('u2'); // John: false
selection.isSelected('u3'); // Harry: false
...
selection.isGroupSelected('adult'); // true
selection.isGroupSelected('teen'); // false
selection.isGroupSelectedAll('adult'); // false
selection.isGroupSelectedAll('teen'); // false

selection.select('u3'); // select Harry. James still selected because Harry and James are on the same age group (adult).

selection.isSelected('u0'); // James: true
selection.isSelected('u1'); // Mike: false
selection.isSelected('u2'); // John: false
selection.isSelected('u3'); // Harry: true
...
selection.isGroupSelected('adult'); // true
selection.isGroupSelected('teen'); // false
selection.isGroupSelectedAll('adult'); // true
selection.isGroupSelectedAll('teen'); // false

selection.select('u2'); // select John. James and Harry will automatically deselected since John is on teen age group.

selection.isSelected('u0'); // James: false
selection.isSelected('u1'); // Mike: false
selection.isSelected('u2'); // John: true
selection.isSelected('u3'); // Harry: false
...
selection.isGroupSelected('adult'); // false
selection.isGroupSelected('teen'); // true
selection.isGroupSelectedAll('adult'); // false
selection.isGroupSelectedAll('teen'); // false

selection.select('u1'); // select Mike. John still selected because Mike and John are on the same age group (teen).

selection.isSelected('u0'); // James: false
selection.isSelected('u1'); // Mike: true
selection.isSelected('u2'); // John: true
selection.isSelected('u3'); // Harry: false
...
selection.isGroupSelected('adult'); // false
selection.isGroupSelected('teen'); // true
selection.isGroupSelectedAll('adult'); // false
selection.isGroupSelectedAll('teen'); // true
```

Select and unselect multiple group:
```dart
selection.selectGroup('adult'); // select all adult.

selection.isGroupSelected('adult'); // true
selection.isGroupSelectedAll('adult'); // true

selection.unselectGroup('adult'); // unselect all adult.

selection.isGroupSelected('adult'); // false
selection.isGroupSelectedAll('adult'); // false
```

Additional methods: see documentation.