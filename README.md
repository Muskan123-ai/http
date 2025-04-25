import 'dart:math';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

void main() {
  runApp(const ApiFetchApp());
}

class ApiFetchApp extends StatelessWidget {
  const ApiFetchApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Random Post Fetcher',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const PostFetcherPage(),
    );
  }
}

class PostFetcherPage extends StatefulWidget {
  const PostFetcherPage({Key? key}) : super(key: key);

  @override
  _PostFetcherPageState createState() => _PostFetcherPageState();
}

class _PostFetcherPageState extends State<PostFetcherPage> {
  Map<String, dynamic>? _post;
  bool _isLoading = false;
  String? _error;

  Future<void> fetchPost() async {
    setState(() {
      _isLoading = true;
      _error = null;
      _post = null;
    });

    final int randomId = Random().nextInt(100) + 1;
    final url = Uri.parse('https://jsonplaceholder.typicode.com/posts/$randomId');

    try {
      final response = await http.get(url);

      if (response.statusCode == 200) {
        setState(() {
          _post = json.decode(response.body);
        });
      } else {
        setState(() {
          _error = 'Failed to load post';
        });
      }
    } catch (e) {
      setState(() {
        _error = 'Error: $e';
      });
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Fetch Random Post'),
      ),
      body: Center(
        child: _isLoading
            ? const CircularProgressIndicator()
            : _error != null
                ? Text(_error!)
                : _post != null
                    ? Padding(
                        padding: const EdgeInsets.all(16.0),
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Text(
                              _post!['title'],
                              style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
                            ),
                            const SizedBox(height: 10),
                            Text(_post!['body']),
                          ],
                        ),
                      )
                    : const Text('Press the button to fetch a post'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: fetchPost,
        child: const Icon(Icons.download),
        tooltip: 'Fetch Post',
      ),
    );
  }
}
