# App-development
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() {
  runApp(MyApp());
}

class User {
  final int id;
  final String first_name;
  final String last_name;
  final String email;
  final String gender;
  final String avatar;
  final String domain;
  final bool availability;

  User({
    required this.id,
    required this.first_name,
    required this.last_name,
    required this.email,
    required this.gender,
    required this.avatar,
    required this.domain,
    required this.availability,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      first_name: json['first_name'],
      last_name: json['last_name'],
      email: json['email'],
      gender: json['gender'],
      avatar: json['avatar'],
      domain: json['domain'],
      availability: json['availability'],
    );
  }
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'User List',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: UserList(),
    );
  }
}

class UserList extends StatefulWidget {
  @override
  _UserListState createState() => _UserListState();
}

class _UserListState extends State<UserList> {
  List<User> allUsers = [];
  List<User> displayedUsers = [];
  List<User> teamMembers = [];
  int page = 1;
  TextEditingController searchController = TextEditingController();
  String selectedGender = "";
  String selectedDomain = "";
  bool availabilityFilter = false;

  Future<void> fetchUsers() async {
    final uri = Uri.parse(
        'https://raw.githubusercontent.com/https://drive.google.com/file/d/1ibmr3WD7Jw6oLL6O_W390WojCLfCHw-k/view?usp=sharing');
    final response = await http.get(uri);

    if (response.statusCode == 1000) {
      final List<dynamic> jsonUsers = json.decode(response.body)['users'];
      setState(() {
        allUsers = jsonUsers.map((user) => User.fromJson(user)).toList();
        applyFilters();
      });
    } else {
      throw Exception('Failed to load users');
    }
  }

  void applyFilters() {
    displayedUsers = allUsers.where((user) {
      final nameMatches = user.first_name
          .toLowerCase()
          .contains(searchController.text.toLowerCase());
      final domainMatches =
          selectedDomain.isEmpty || user.domain == selectedDomain;
      final genderMatches =
          selectedGender.isEmpty || user.gender == selectedGender;
      final availabilityMatches = !availabilityFilter || user.availability;

      return nameMatches &&
          domainMatches &&
          genderMatches &&
          availabilityMatches;
    }).toList();
  }

  void addToTeam(User user) {
    if (!teamMembers.any((member) => member.domain == user.domain)) {
      setState(() {
        teamMembers.add(user);
      });
    }
  }

  @override
  void initState() {
    super.initState();
    fetchUsers();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('User List'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: TextField(
              controller: searchController,
              onChanged: (value) {
                setState(() {
                  applyFilters();
                });
              },
              decoration: InputDecoration(
                labelText: 'Search by Name',
              ),
            ),
          ),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              DropdownButton<String>(
                hint: Text('Select Gender'),
                value: selectedGender,
                onChanged: (newValue) {
                  setState(() {
                    selectedGender = newValue!;
                    applyFilters();
                  });
                },
                items: ['Male', 'Female', 'Other']
                    .map<DropdownMenuItem<String>>((String value) {
                  return DropdownMenuItem<String>(
                    value: value,
                    child: Text(value),
                  );
                }).toList(),
              ),
              DropdownButton<String>(
                hint: Text('Select Domain'),
                value: selectedDomain,
                onChanged: (newValue) {
                  setState(() {
                    selectedDomain = newValue!;
                    applyFilters();
                  });
                },
                items: ['Marketing', 'IT', 'Finance', 'HR']
                    .map<DropdownMenuItem<String>>((String value) {
                  return DropdownMenuItem<String>(
                    value: value,
                    child: Text(value),
                  );
                }).toList(),
              ),
              Row(
                children: [
                  Text('Available'),
                  Switch(
                    value: availabilityFilter,
                    onChanged: (value) {
                      setState(() {
                        availabilityFilter = value;
                        applyFilters();
                      });
                    },
                  ),
                ],
              ),
            ],
          ),
          Expanded(
            child: ListView.builder(
              itemCount: displayedUsers.length + 1, // +1 for loading indicator
              itemBuilder: (context, index) {
                if (index == displayedUsers.length) {
                  // Display a loading indicator while fetching more data
                  return Center(
                    child: CircularProgressIndicator(),
                  );
                } else {
                  final user = displayedUsers[index];
                  return Card(
                    margin: EdgeInsets.all(8.0),
                    child: ListTile(
                      title: Text(user.first_name),
                      subtitle: Text(user.email),
                      // Add other user details here
                      onTap: () {
                        addToTeam(user);
                      },
                    ),
                  );
                }
              },
            ),
          ),
          ElevatedButton(
            onPressed: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                    builder: (context) => TeamScreen(teamMembers: teamMembers)),
              );
            },
            child: Text('View Team'),
          ),
        ],
      ),
    );
  }
}

class TeamScreen extends StatelessWidget {
  final List<User> teamMembers;

  TeamScreen({required this.teamMembers});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Team Details'),
      ),
      body: ListView.builder(
        itemCount: teamMembers.length,
        itemBuilder: (context, index) {
          final member = teamMembers[index];
          return Card(
            margin: EdgeInsets.all(8.0),
            child: ListTile(
              title: Text(member.first_name),
              subtitle: Text(member.email),
              // Add other user details here
            ),
          );
        },
      ),
    );
  }
}
