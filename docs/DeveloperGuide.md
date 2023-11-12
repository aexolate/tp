---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

* Some code adapted from http://code.makery.ch/library/javafx-8-tutorial/ by Marco Jakob

* Some code inspired by https://github.com/se-edu/addressbook-level4 by CS2103T Teaching Team

* Copyright by Susumu Yoshida - http://www.mcdodesign.com/
  * address_book_32.png

* Copyright by Jan Jan Kovařík - http://glyphicons.com/
  * calendar.png

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document `docs/diagrams` folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</div>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person and to commit the current state of the addressbook to be used for `undo` and `redo` commands).
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered and sorted_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)


### Storage component

**API** : [`Storage.java`](https://github.com/AY2324S1-CS2103T-T14-1/tp/blob/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### Add feature

#### Implementation

The add feature is facilitated by the following operations:
* `AddCommandParser#parse` — Parses user's inputs and checks that inputs are valid.
* `model#hasPerson` — Checks if the employee exists in EmployeeManager.
* `model#addPerson` — Adds an employee to EmployeeManager.

Given below is an example usage scenario and how the add mechanism behaves at each step.

Step 1. The user enters `add i/A00001 n/John Doe p/98765432 e/johnd@example.com d/Finance r/Manager s/5000` to add a new employee `John Doe`.
The user input is parsed and checked using `AddCommandParser#parse` to create an `AddCommand` that contains the new employee.

Step 2: `AddCommand` then checks that the new employee is not a duplicate employee using `model#hasPerson` before
adding the new employee into EmployeeManager using `model#addPerson`.

The following activity diagram summarizes what happens when a user executes the add command:

<img src="images/AddActivityDiagram.png" width="425" />

#### Design considerations

Aspect: What is considered a duplicate employee:
* **Alternative 1:** Check if the name of the given employee matches the name of another employee in EmployeeManager.
  * Pros: Only need to check for duplicate names.
  * Cons: Employees can have the same names so it is not realistic to restrict the addition of employees to only employees with different names.
* **Alternative 2 (current choice):** Check if the ID of an employee matches the ID of another employee in EmployeeManager.
  * Pros: Able to store employees with the same attributes if their IDs are different.
  * Cons: Need to store an additional field for ID.

### History feature
The `history` feature allows users to access past valid commands that they made. This feature is supported by 
three classes, `CommandHistory`, `HistoryCommandParser` and `HistoryCommand`. 

#### Implementation

`CommandHistory`: This class is responsible for storing the history of valid commands that the user has entered.  
`HistoryCommandParser`: Parses the user input to create an appropriate HistoryCommand object.  
`HistoryCommand`: Executes the history command.  

After a successful user command has been executed `CommandHistory#add(input)` will be called to store the successful 
user input in the `CommandHistory`. When a user inputs a inputs a history command with the appropriate argument, the 
`AddressBookParser` will be called to produce the `HistoryCommandParser` to properly parse the input. A 
`HistoryCommand` object will be created. When the `Command#execute(model, commandHistory)` is called on the 
`HistoryCommand`, the `HistoryCommand` will call `CommandHistory#getUserCommandHistory()` to get the list of
valid user commands. `HistoryCommand` will then return a string of the appropriate number of user command that the
user has specified.

![History Command Class Diagram](images/HistoryCommandClassDiagram.png)


#### Design Considerations

**Aspect: Which component should we choose to store CommandHistory:**

* **Alternative 1 (current choice):** Store it under `Logic`.  
**Why:** The `Logic` component is responsible for command execution and parsing. 
Adding a history feature here would allow you to easily keep track of commands as they are executed.

* **Alternative 2:** Store it under `Model`.  
**Why:** The `Model` component is responsible for maintaining the state of the application. If we consider
the command history part of the application's state, it might make more sense to put it here.

**Decision:** Since the command history is only active for the duration of the application and does not need
to be saved, placing it in the `logic` component is more appropriate. If the command history need to persist
across sessions, it might be better to place it in the `model` component, as this layer is generally responsible
for data to be saved.

### Sort feature

#### Implementation

The sort command is facilitated by `ModelManager`. `ModelManager` contains a `SortedList<Person>` and 
`FilteredList<Person>` to provide sorting and filtering on the same list. The original list is fetched from 
`AddressBook#getPersonList()` and passed to the `FilteredList`. The `FilteredList` is then passed to the `SortedList`.
By returning the `SortedList` as an `ObservableList<Person>`, the UI will be able to update the person cards displayed
after the `find` or `sort` command has been used.

To facilitate comparing between two `Person` objects, the attributes also implement `Comparable<Person>`. The
comparators for the attributes can be used to then compare two Person on a specific attribute.

The command only allows for the sorting to be performed on one of the Person's attribute. The `SortCommandParser` will
extract the attribute, and create a `SortCommand` using the corresponding comparator for that attribute. The sequence 
diagram below illustrates how the comparator is obtained and passed to the Model to be executed.

<img src="images/SortSequenceDiagram.png" width="574" />

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `SortCommandParser` should
end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

  
### Find Feature

#### Implementation

The Find feature is facilitated by the following operations:
* `Model#updateFilteredPersonList()` — Updates the filtered person list based on the Predicate<Person> given to the FindCommand object.

Given below is an example usage scenario and how the find mechanism behaves at each step.

Step 1. The user executes `find n/alex bernice` to find all employees that either contain the full word `alex` or `bernice` in their name. 
`FindCommandParser` parses the user input and creates a `FindCommand`.

Step2. Then `FindCommand` is executed, and it will call `Model#updateFilteredPersonList()` to update the filtered list in the model to only include
employees whose names contain either the full word `alex` or `bernice`.

The following sequence diagram illustrates how the find feature works:
![FindSequenceDiagram](images/FindSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `FindCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of the diagram.

</div>

### Edit Feature
The `edit` feature allows users to update specific information about certain employees. This feature is supported by 
three classes, `EditCommand`, `EditCommandParser` and `Model`.

#### Implementation

`EditCommand`: Stores the edited changes to be changed.<br>
`EditCommandParser`: Parses the user's edit command to create an appropriate EditCommand object.<br>
`Model`: Fetches the employee list from `Model#getSortedFilteredPersonList()` and finds the employee whose information needs to be edited.

Given below is an example usage scenario of edit command. 

Step 1: The user enters `edit 2 p/12345678` to update the phone number of the second employee in the list.<br>
`EditCommandParser` parses the user command and creates a `EditCommand`.

Step 2: `EditCommand` then gets executed. It calls `Model#setPerson()` to update the information of the chosen employee.

The sequence diagram below illustrates how the edit command works: 
![EditCommandSequenceDiagram](images/EditCommandSequenceDiagram.png)

### Increment Feature

#### Implementation

The increment feature is facilitated by the following operations:
* `Model#getSortedFilteredPersonList()` — Returns the list of employees in the filtered list.
* `Model#setPerson(target, editedPerson)` — Replaces the given `target` with the `editedPerson` that has the incremented salary with all other attributes of `target` unchanged.

Given below is an example usage scenario and how the increment mechanism behaves at each step.

Step 1. The user has executed `find d/Marketing` to filter the employee list by the department `Marketing`.
* The `FindCommand` updates the filtered list in `Model` to contain only employees whose department is `Marketing`.

Step 2. The user executes `increment 1000` to increment the salaries of all employees in the filtered list.
The associated command `IncrementCommand` first calls `Model#getSortedFilteredPersonList()` to obtain the filtered list of persons.

Step 3. `IncrementCommand` then checks that the increment is valid for all persons in the filtered list using `IncrementCommand#checkValidIncrement(personList)`.

Step 4. For each person in the filtered list, an `editedPerson` with the incremented salary and no other details changed is constructed, before `Model#setPerson(target, editedPerson)` is called to replace the current person with the `editedPerson`.

The following sequence diagram illustrates how the increment feature works:
![IncrementSequenceDiagram](images/IncrementSequenceDiagram.png)

#### Design considerations

**Aspect: When the validity of the given `increment` is checked:**

* **Alternative 1 (current choice):** Check validity before incrementing any person’s salary.
  * Pros: Ensures that `increment` is valid before any modifications is made to the persons in EmployeeManager.
  * Cons: Have to loop through the filtered list twice: once to check the validity of `increment` and once to increment the salaries.

* **Alternative 2:** Check validity while incrementing each person’s salary.
  * Pros: Less time required to check and increment salaries.
  * Cons: If the `increment` is invalid for a person halfway through the list, some persons would have their salaries incremented while the remaining persons would not have their salaries incremented.

### Undo/redo feature

#### Implementation

The undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history and remove undone states.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how the undo operation works:

![UndoSequenceDiagram](images/UndoSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

<img src="images/CommitActivityDiagram.png" width="250" />

#### Design considerations

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

### Clear Feature

The `clear` feature is used to clear all persons from the displayed list in EmployeeManager.

#### Implementation
When the `clear` command is executed, it will call `Model#clearSortedFilteredPersonList`.
`Model#clearSortedFilteredPersonList` will loop through the persons in the filtered list and
delete each person from the filtered list using `Model#deletePerson` until the list is cleared.

![ClearCommandSequenceDiagram](images/ClearCommandSequenceDiagram.png)


### Export/Import Feature

The `export` feature allows the data file to be saved as a `.json` file on the disk. The `import` feature allows a
data file to be imported from the disk, replacing the current data in the application.

#### Implementation

The import and export commands both use `JsonAddressBookStorage` to facilitate writing and reading of data files.

The import function calls `JsonAddressBookStorage#readAddressBook` which will read the file in the specified file path
and attempt to parse the JSON data into a `ReadOnlyAddressBook`. The function also checks if the data file is invalid,
such as if it contains illegal value, or if no file is found at the file path.

The export function calls `JsonAddressBookStorage#saveAddressBook` which will serialize a `ReadOnlyAddressBook` and
write it to the file path on the disk.

A GUI option for importing and exporting was also implemented to allow users who are unfamiliar with file paths to use
the feature as well.

Another implementation detail is creating an abstract class `FileCommand` which inherits `Command`. Both `ImportCommand`
and `ExportCommand` inherit `FileCommand`. The reason for creating the `FileCommand` class is to reduce code 
duplication as both commands share the same logic for checking the validity of the `Path` received.

![FileCommandClassDiagram](images/FileCommandClassDiagram.png)

#### Design considerations

**Aspect: Format of the export & import command:**

* **Alternative 1 (current choice):** `export FILE_PATH`, `import FILE_PATH`.
    * Pros: Provides flexibility in where the file can be exported to and imported from.
    * Cons: Not all users might be familiar with specifying file paths. However, it can be mitigated
      with a GUI option to select the file path.

* **Alternative 2:** `export FILE_NAME`, `import FILE_NAME`.
    * Pros: Simple to use, only have to specify file name and not be concerned with file path.
    * Cons: Troublesome, the files must be exported and imported from the same directory as the application.

--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* has a need to manage a significant number of employees
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: The application streamlines the process of handling 
large-scale employee data. With rapid typing capabilities, users
can efficiently input and update employee details, reducing time
spent on data entry tasks.

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                             | I want to …​                                                                                              | So that ...                                                                                 |
|----------|-------------------------------------|-----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| `*`      | potential user exploring the app    | view the app populated with sample data                                                                   | I can easily see how the app will look when it is in use                                    |
| `* *`    | user paranoid about losing progress | have the data automatically saved                                                                         | I will not lose any progress when the application shuts down unexpectedly                   |
| `* *`    | new user                            | clear all current data                                                                                    | I can get rid of experimental data I used for exploring the app                             |
| `*`      | new user                            | view a user-friendly tutorial of the app                                                                  | I can understand the app's features                                                         |
| `* * *`  | HR staff member                     | add an employee to the application                                                                        | I can keep track of their details                                                           |
| `*`      | HR staff member                     | view two employees' details side by side                                                                  | I can easily compare the differences of the employees                                       |
| `* * *`  | HR staff member                     | edit employees' details                                                                                   | I can keep the details of the employees up-to-date                                          |
| `* * *`  | HR staff member                     | remove employees who have left the company                                                                | their data does not clutter the application                                                 |
| `* * *`  | HR staff member                     | search for employees using their names                                                                    | I can find the employee that I am looking for                                               |
| `* * *`  | HR staff member                     | search for employees based on their departments                                                           | I can obtain the contact information of the employees in a department                       |
| `* * *`  | HR staff member                     | search for employees based on their roles                                                                 | I can easily increment the pay of all the staff with that role efficiently                  |
| `* * *`  | HR staff member                     | search for employees using multiple attributes (e.g. department, salary, etc.)                            | I can narrow down my search to a target group of employees                                  |
| `*`      | HR staff member                     | view who I have looked at recently when I click the search bar                                            | I can easily fetch employees that I have analyzed recently and make my work more convenient |
| `*`      | HR staff member                     | generate reports on employees' performance                                                                | I can provide the upper management better insights into their employees                     |
| `* *`    | expert user                         | combine functions together                                                                                | I can save time on commonly performed tasks that requires more than one function            |
| `* *`    | expert user                         | bind keyboard shortcuts for certain frequently used functions                                             | I can work more efficiently                                                                 |
| `* * *`  | HR staff member                     | order my search results by various attributes (e.g. name, department, etc.)                               | I can view a list of employees in my desired manner                                         |
| `* * *`  | long-time user                      | remove unused employee data                                                                               | I am not distracted by irrelevant data                                                      |
| `* *`    | HR staff member                     | view the history of changes, including who made the change and when the change was made                   | I can use them for auditing purposes                                                        |
| `* *`    | HR staff member                     | add notes to employees' details                                                                           | I can document important information and interactions                                       |
| `*`      | advanced user                       | edit the saved data files manually without using the application                                          | I can update and organise new information more conveniently                                 |
| `*`      | user with poor eyesight             | view the information without straining my eyes by adjusting the text size                                 | it is not difficult for me to use the application                                           |
| `*`      | HR staff member                     | collect and view an analysis of diversity metrics                                                         | I can promote a more diverse workplace                                                      |
| `* *`    | HR staff member                     | export and import employee details                                                                        | I can create backups and recover from data corruption                                       |
| `*`      | HR staff member                     | set performance goals for employees                                                                       | I can align the employees with the company’s objectives                                     |
| `* *`    | careless HR staff member            | undo my last command                                                                                      | I can easily revert to a copy with no mistakes if I made mistakes                           |
| `* *`    | careless HR staff member            | redo my command                                                                                           | I can redo a command that I accidentally undid                                              |
| `* *`    | HR staff member                     | increment and decrement multiple employees' salaries together by the same amount                          | I do not need to repeat the same action for multiple employees                              |

### Use cases

(For all use cases below, the **System** is the `EmployeeManager` and the **Actor** is the `user`, unless specified otherwise)

**Use case: UC1 - View all employees**

**MSS**

1. User requests to view all employees.
1. EmployeeManager shows a list of employees.
1. Use case ends.

**Use case: UC2 - Add a new employee**

**MSS**

1. User requests to add a new employee with employee details.
1. EmployeeManager adds the employee.
1. EmployeeManager shows an updated list of employees.
1. Use case ends.

**Extensions**

* 1a. EmployeeManager detects that not all required information is entered.
  * 1a1. EmployeeManager informs user that some information is missing.
  * 1a2. EmployeeManager requests user to enter details of the employee.
  * 1a3. User enters the requested details.
  * Steps 1a1 - 1a3 are repeated until the data entered are correct.
  * Use case resumes from step 2.
* 1b. EmployeeManager detects invalid information.
  * 1b1. EmployeeManager informs user that some information is invalid.
  * 1b2. EmployeeManager requests user to enter details of the employee.
  * 1b3. User enters the requested details.
  * Steps 1b1 - 1b3 are repeated until the data entered are correct.
  * Use case resumes from step 2.
* 1c. EmployeeManager detects a duplicate employee.
  * 1c1. EmployeeManager informs user that the employee already exists.
  * Use case ends.

**Use case: UC3 - Delete an employee**

**MSS**

1. User requests to delete an employee with an index.
1. EmployeeManager deletes the employee.
1. EmployeeManager shows an updated list of employees.
1. Use case ends.

**Extensions**

* 1a. The given index is invalid.
  * 1a1. EmployeeManager informs user that the given index is invalid.
  * 1a2. User enters new index.
  * Steps 1a1-1a2 are repeated until the index entered is valid.
  * Use case resumes from step 2.

**Use case: UC4 - Find an employee**

**MSS**

1. User requests to find an employee with name and surname
2. EmployeeManager finds all employees whose names contain either the full name or surname keyword 
3. EmployeeManager updates the displayed list with the found employees.
3. Use case ends.

**Extensions**

* 1a. The user did not use a valid predefined prefix to search for employee by name
    * 1a1. EmployeeManager informs user that the input is invalid.
    * 1a2. User enters new input and new prefix.
    * Steps 1a1-1a2 are repeated until the input is valid.
    * Use case resumes from step2.

**Use case: UC5 - Bulk increment salaries**

**MSS**

1. User requests to increment salaries of all employees in the displayed list by an increment amount.
1. EmployeeManager increments the salaries of all employees in the displayed list.
1. EmployeeManager shows an updated list of employees.
1. Use case ends.

**Extensions**

* 1a. The given increment amount is invalid.
    * 1a1. EmployeeManager informs user that the increment amount is invalid.
    * 1a2. User enters new increment amount.
    * Steps 1a1-1a2 are repeated until the increment amount entered is valid.
    * Use case resumes from step 2.

**Use case: UC6 - View Command History**

**MSS**

1. User requests to view the command history of a certain number of past commands.
1. EmployeeManager fetches the required number of recent valid commands from the CommandHistory.
1. EmployeeManager displays the fetched commands to the user.
1. Use case ends.

**Extensions**

* 1a. The user enters an invalid number.
    * 1a1. EmployeeManager informs user that the number is invalid.
    * 1a2. User enters history command with number.
    * Steps 1a1-1a2 are repeated until the number entered is valid.
    * Use case resumes from step 2.

**Use case: UC7 - Clear the displayed list**

**MSS**

1. User requests clear the displayed list.
1. EmployeeManager clears the displayed list.
1. EmployeeManager displays an empty list.
1. Use case ends.

**Use case: UC8 - Exporting the data file**

**MSS**

1. User requests to export the data file to a specified file path.
1. EmployeeManager exports the data file to the specified file path.
1. EmployeeManager displays a success message.
1. Use case ends.

**Extensions**

* 1a. The user enters an invalid file path.
    * 1a1. EmployeeManager informs user that the file path is invalid.
    * 1a2. User enters a new file path.
    * Steps 1a1-1a2 are repeated until the file path is valid.
    * Use case resumes from step 2.

**Use case: UC9 - Importing the data file**

**MSS**

1. User requests to import the data file from a specified file path.
1. EmployeeManager imports the data file from the specified file path.
1. EmployeeManager shows the updated list of employees imported from the data file.
1. Use case ends.

**Extensions**

* 1a. The user enters an invalid file path.
    * 1a1. EmployeeManager informs user that the file path is invalid.
    * 1a2. User enters a new file path.
    * Steps 1a1-1a2 are repeated the file path is valid.
    * Use case resumes from step 2.
* 1b. The user enters a file path where the file doesn't exist.
    * 1b1. EmployeeManager informs user that the file does not exist at the file path.
    * 1b2. User enters a new file path.
    * Steps 1a1-1a2 are repeated until the file path has an existing file.
    * Use case resumes from step 2.
* 1c. The user enters a file path where the data file is corrupted.
    * 1c1. EmployeeManager informs user that the data file is corrupted.
    * 1c2. User enters a new file path.
    * Steps 1c1-1c2 are repeated until the file path has a valid data file.
    * Use case resumes from step 2.

### Non-Functional Requirements

1.  **Cross-platform Capability**: Should work on any _mainstream OS_ as long as it has Java `11` or above installed.
2.  **Storage Capacity**: Should be able to store up to `200` persons without a noticeable sluggishness in performance for typical usage.
3.  **UI Responsiveness**: The UI should respond to user interaction. 
4.  **Error Handling**: Should gracefully handle input errors without system crash and data loss.
5.  **Code Documentation**: Code should be well-documented to facilitate maintenance and updates.
6.  **Code Version Control**: The software codebase should be managed using _Git Milestone_ to track changes.
7.  **Keyboard Efficiency**: A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, OS-X.
* **UI**: User Interface, the graphical or textual interface through which a user interacts with a software application.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

### Adding an employee

1. Adding an employee to the displayed list

    1. Test case: `add i/A00001 n/John Doe p/98765432 e/johnd@example.com d/Finance r/Manager s/5000`<br>
       Expected: An employee with the correct details is added to the end of the list.

    1. Test case: `add`<br>
       Expected: No employee is added. Error details shown in the status message.


### Deleting an employee

1. Deleting an employee from the displayed list

   1. Prerequisites: List all employees using the `list` command. Multiple employees in the list.

   1. Test case: `delete 1`<br>
      Expected: First employee is deleted from the list. Details of the deleted employee shown in the status message.

   1. Test case: `delete 0`<br>
      Expected: No employee is deleted. Error details shown in the status message.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

### Incrementing salaries of multiple employees

1. Incrementing salaries of all employees in the displayed list of employees.

    1. Test case: `increment 1000`<br>
       Prerequisite: Salaries of all employees do not exceed the maximum salary after increasing by 1000. <br>
       Expected: Salaries of all employees in the list increased by 1000.

    1. Test case: `increment -10000` <br>
       Prerequisite: Salary of at least one employee in the displayed list is below 10000. <br>
       Expected: No change in salaries of all employees. Error details shown in the status message.

### Undoing a modification

1. Undo the previous command that caused a modification in employee data.

    1. Test case: `delete 1` followed by `undo`<br>
       Prerequisite: List all persons using the `list` command. At least one person in the list.<br>
       Expected: The list is reverted to the list of employees before `delete 1` was executed.

    1. Test case: `undo`<br>
       Prerequisite: All commands have been undone or no commands that made a modification to employee data has been made.<br>
       Expected: No change in displayed list and employee data. Error details shown in the status message.
        1. Tip: A quick way to achieve the prerequisite is to close and reopen the application.

### Redoing the previous undone command

1. Redo the previous undone command caused by `undo`.

    1. Test case: `delete 1` followed by `undo` followed by `redo`<br>
       Prerequisite: List all persons using the `list` command. At least one person in the list.<br>
       Expected: The list is reverted to the list of employees after `delete 1` was executed.

    1. Test case: `redo`<br>
       Prerequisite: No commands have been undone.<br>
       Expected: No change in displayed list and employee data. Error details shown in the status message.

### Editing an employee

1. Editing an employee in the displayed list of employees.

    1. Prerequisites: List all employees using the `list` command. At least one employee in the list.

    1. Test case: `edit 1 n/John Tan`<br>
       Expected: First employee in the list has their name edited to `John Tan`. Details of the edited employee is shown in the status message.

    1. Other correct edit commands to try: `edit 1 p/12345678`, `edit 1 d/Accounting` and `edit 1 r/Manager`<br>
       Expected: Similar to previous, the respective fields gets edited.

    1. Test case: `edit 0`<br>
       Expected: No employee is edited. Error details shown in the status message.

    1. Other incorrect edit commands to try: `edit`, `edit x`, `...` (where x is larger than the list size)<br>
       Expected: Similar to previous.       

### Sorting the list

1. Sorting the displayed list of employees.

    1. Prerequisites: List all employees using the `list` command. Multiple employees in the list.

    1. Test case: `sort n/`<br>
       Expected: Displayed list is sorted by alphabetical order of the name field.

    1. Other correct sort commands to try: `sort d/`, `sort r/` and `sort e/`<br>
       Expected: Similar to previous, the respective fields gets sorted based on lexicographical order.

    1. Test case: `sort`<br>
       Expected: Displayed list is not sorted. Error details shown in the status message.

### Finding an employee

1. Finds an employee by the given field.

    1. Tip: Create distinct employees, one with the name `John`, one with the department `Finance`, 
   and one with the role `Manager` to effectively test whether the find command works for each field.

    1. Test case: `find n/John`<br>
       Expected: Finds all employees whose names include the word `John`.

    1. Test case: `find d/Finance`<br>
       Expected: Finds all employees whose department include the word `Finance`.

    1. Test case: `find r/Manager`<br>
      Expected: Finds all employees whose role include the word `Manager`.

    1. Test case: `find`<br>
      Expected: The displayed list is not filtered. Error details shown in the status message.

### Viewing command history

1. Viewing the command history.

    1. Test case: `delete 1` followed by `history 1`<br>
       Expected: The previous command `delete 1` is shown in the status message.

    1. Test case: `history 0`<br>
       Expected: Error details shown in the status message.

### Exporting data file

1. Exporting the data file.

    1. Test case: `export fy2324.json`<br>
       Prerequisite: The application has write access to the directory where EmployeeManager is.<br>
       Expected: The data file will be written to the directory where EmployeeManager is as `fy2324.json`.

    1. Test case: `export fy2324`<br>
       Expected: The file name does not have the .json file extension. Error details shown in the status message.

   1. Test case: `export /`<br>
      Expected: The file path is invalid as it is a directory. Error details shown in the status message.


### Importing data file

1. Importing the data file.

    1. Test case: `export fy2324.json`, followed by `clear`, then `import fy2324.json`<br>
       Prerequisites: The application has read access to the specified file path. The file `fy2324.json` must exist
       at the specified file path - which will be the directory where EmployeeManager is in this case.<br>
       Expected: The data file `fy2324.json` will be imported into the application and overwrite existing data.

    1. Test case: `import fy9999.json`<br>
       Prerequisites: The file `fy9999.json` does not exist in the directory where EmployeeManager is.<br>
       Expected: Error details shown in the status message.

   1. Test case: `export fy2324.json`, followed by removing some of the brackets in the exported `fy2324.json` file 
      using any text editor, followed by `import fy2324.json`.<br>
      Prerequisites: The file `fy2324.json` is corrupted (the data does not conform to the format of EmployeeManager's 
      data file).<br>
      Expected: Error details shown in the status message.

## **Appendix: Planned Enhancements**

### Export command FILENAME should be a non-empty String

The current implementation of `export` command allows the use of an empty string as the FILENAME, allowing commands like `export .json` to execute successfully.
While this behaviour is technically allowed on windows, it has led to inconsistencies on other operating systems, 
where no file is created despite the command executing successfully. To address this, we plan to make the validation checks stricter for the 
FILENAME in the `export` command. This enhancement will prevent the use of empty strings in FILENAME, 
thereby reducing the chances of bugs and ensuring a more consistent experience across different operating systems.

### Allow the use of s/o in name field

The current implementation of EmployeeManager designates `s/` as a prefix for salary. As a result, when the name field includes `s/o`, it triggers 
several errors such as those related to multiple prefixes or salary constraints. 

![PlannedEnhancementSOError](images/PlannedEnhancementSOError.png)

We plan to add an escape character, such as a backslash, "\", This will allow more flexibility for name entries. For example, with this enhancement, a command like `edit 1 n/John \s/o dep` would be valid,
changing the name of the person at index one to `John s/o dep`. The `s/o` will be recognised as part of a name rather than a salary prefix.

### Change all mentions of addressbook to employeemanager

Our project is a brownfield project adapted from AB3, it contains several references to addressbook such as the `AddressBookParser` and `AddressBook`. 
While our primary focus has been on enhancing and improving the functionality of the software to cater to our target audience, we recognise the 
need to have consistent naming conventions for clarity. Therefore, we plan to update all mentions of `addressbook` to `employeemanager`.

### Stringent email validation

The current implementation of EmployeeManager allows invalid emails such as `abc@gmail`, which fall short of the standard email address formats. 
We understand the importance of data validation to our target audience, hence we plan add more stringent email validation that checks for proper
`username@domainname.extension` formatting.

### Improve clarity of success message for `undo` commands

The current implementation of the `undo` command only shows that the command has been undone successfully, it does not specify what command has been undone. 

![PlannedEnhancementUndoSuccessMessage](images/PlannedEnhancementUndoSuccessMessage.png)

To enhance user experience and provide detailed feedback to the user, we plan to include the specific command that was undone.

### Improve clarity of success message for `redo` commands

The current implementation of the `redo` command only shows that the command has been redone successfully, it does not specify what command has been redone.

![PlannedEnhancementRedoSuccessMessage](images/PlannedEnhancementRedoSuccessMessage.png)

To enhance user experience and provide detailed feedback to the user, we plan to include the specific command that was redone.

### Enhance validation for `sort` command

The current implementation of the `sort` command allows users to input additional values after the sorting prefix, although they will be ignored. 
For example commands like `sort n/abc` and `sort n/s/` are both valid sort commands that will sort by name. 
We plan to enhance validation such that an error message will be shown if extra values are input after the sorting prefix.
The proposed error message, `The sort command prefixes should not be used with values`, will guide the user towards the correct usage of the `sort` command.
