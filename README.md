import 'package:flutter/material.dart';
import 'dart:convert'; // For JSON decoding
import 'package:http/http.dart' as http; // For API requests

void main() {
  runApp(SoccerApp());
}

class SoccerApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Soccer App',
      theme: ThemeData(
        primarySwatch: Colors.green,
      ),
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  List<dynamic> teams = [];
  bool isLoading = true;

  @override
  void initState() {
    super.initState();
    fetchTeams();
  }

  Future<void> fetchTeams() async {
    const String apiUrl = 'https://api.football-data.org/v4/competitions/PL/teams';
    const String apiKey = 'YOUR_API_KEY_HERE'; // Replace with your API key

    try {
      final response = await http.get(
        Uri.parse(apiUrl),
        headers: {'X-Auth-Token': apiKey},
      );

      if (response.statusCode == 200) {
        final data = json.decode(response.body);
        setState(() {
          teams = data['teams'];
          isLoading = false;
        });
      } else {
        throw Exception('Failed to load teams');
      }
    } catch (e) {
      setState(() {
        isLoading = false;
      });
      print('Error: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Top Soccer Teams'),
      ),
      body: isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: teams.length,
              itemBuilder: (context, index) {
                final team = teams[index];
                return ListTile(
                  leading: Image.network(
                    team['crest'] ?? '',
                    width: 40,
                    height: 40,
                    errorBuilder: (context, error, stackTrace) {
                      return Icon(Icons.sports_soccer);
                    },
                  ),
                  title: Text(team['name']),
                  subtitle: Text(team['area']['name']),
                  trailing: Icon(Icons.arrow_forward),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => TeamPage(teamId: team['id']),
                      ),
                    );
                  },
                );
              },
            ),
    );
  }
}

class TeamPage extends StatefulWidget {
  final int teamId;

  TeamPage({required this.teamId});

  @override
  _TeamPageState createState() => _TeamPageState();
}

class _TeamPageState extends State<TeamPage> {
  Map<String, dynamic>? teamDetails;
  bool isLoading = true;

  @override
  void initState() {
    super.initState();
    fetchTeamDetails();
  }

  Future<void> fetchTeamDetails() async {
    const String apiUrl = 'https://api.football-data.org/v4/teams/';
    const String apiKey = 'YOUR_API_KEY_HERE'; // Replace with your API key

    try {
      final response = await http.get(
        Uri.parse('$apiUrl${widget.teamId}'),
        headers: {'X-Auth-Token': apiKey},
      );

      if (response.statusCode == 200) {
        final data = json.decode(response.body);
        setState(() {
          teamDetails = data;
          isLoading = false;
        });
      } else {
        throw Exception('Failed to load team details');
      }
    } catch (e) {
      setState(() {
        isLoading = false;
      });
      print('Error: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(teamDetails?['name'] ?? 'Team Details'),
      ),
      body: isLoading
          ? Center(child: CircularProgressIndicator())
          : Padding(
              padding: const EdgeInsets.all(16.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  if (teamDetails?['crest'] != null)
                    Center(
                      child: Image.network(
                        teamDetails!['crest'],
                        width: 100,
                        height: 100,
                        errorBuilder: (context, error, stackTrace) {
                          return Icon(Icons.sports_soccer, size: 100);
                        },
                      ),
                    ),
                  SizedBox(height: 16),
                  Text(
                    'Team: ${teamDetails?['name']}',
                    style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text(
                    'Country: ${teamDetails?['area']['name']}',
                    style: TextStyle(fontSize: 18),
                  ),
                  SizedBox(height: 8),
                  Text(
                    'Founded: ${teamDetails?['founded'] ?? 'Unknown'}',
                    style: TextStyle(fontSize: 18),
                  ),
                  SizedBox(height: 16),
                  Text(
                    'Players:',
                    style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                  ),
                  Expanded(
                    child: ListView.builder(
                      itemCount: (teamDetails?['squad'] ?? []).length,
                      itemBuilder: (context, index) {
                        final player = teamDetails!['squad'][index];
                        return ListTile(
                          title: Text(player['name']),
                          subtitle: Text(player['position'] ?? 'Unknown Position'),
                          leading: Icon(Icons.person),
                        );
                      },
                    ),
                  ),
                ],
              ),
            ),
    );
  }
}
