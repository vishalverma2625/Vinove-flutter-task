import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'package:intl/intl.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Location Screen',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: LocationScreen(),
    );
  }
}

class LocationScreen extends StatefulWidget {
  @override
  _LocationScreenState createState() => _LocationScreenState();
}

class _LocationScreenState extends State<LocationScreen> {
  GoogleMapController? _controller;
  Set<Marker> _markers = {};
  List<LatLng> _route = [];
  DateTime selectedDate = DateTime.now();
  List<DateTime> visitedDates = [DateTime.now().subtract(Duration(days: 1)), DateTime.now()];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Member Location")),
      body: Column(
        children: [
          Expanded(
            child: GoogleMap(
              initialCameraPosition: CameraPosition(
                target: LatLng(37.7749, -122.4194), // Sample coordinates
                zoom: 14,
              ),
              markers: _markers,
              onMapCreated: (controller) {
                _controller = controller;
              },
              polylines: {
                Polyline(
                  polylineId: PolylineId("route"),
                  points: _route,
                  color: Colors.blue,
                ),
              },
            ),
          ),
          Container(
            padding: EdgeInsets.all(10),
            child: Column(
              children: [
                Text(
                  "Visited Locations",
                  style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                ),
                _buildTimeline(),
                SizedBox(height: 10),
                ElevatedButton(
                  onPressed: () => _selectDate(context),
                  child: Text('Select Date'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildTimeline() {
    return Column(
      children: visitedDates.map((date) {
        return ListTile(
          title: Text(DateFormat('yyyy-MM-dd').format(date)),
          onTap: () => _onDateSelected(date),
        );
      }).toList(),
    );
  }

  Future<void> _selectDate(BuildContext context) async {
    final DateTime? picked = await showDatePicker(
      context: context,
      initialDate: selectedDate,
      firstDate: DateTime(2020),
      lastDate: DateTime.now(),
    );
    if (picked != null && picked != selectedDate) {
      setState(() {
        selectedDate = picked;
        _loadRouteForDate(picked);
      });
    }
  }

  void _onDateSelected(DateTime date) {
    setState(() {
      selectedDate = date;
      _loadRouteForDate(date);
    });
  }

  void _loadRouteForDate(DateTime date) {
    // Load route and markers for the selected date.
    _markers.clear();
    _route.clear();

    // Add dummy route for example
    _route.add(LatLng(37.7749, -122.4194));
    _route.add(LatLng(37.8049, -122.4194));

    _markers.add(Marker(markerId: MarkerId('start'), position: _route.first));
    _markers.add(Marker(markerId: MarkerId('end'), position: _route.last));

    _controller?.animateCamera(CameraUpdate.newLatLngBounds(
      LatLngBounds(
        southwest: _route.first,
        northeast: _route.last,
      ),
      50,
    ));
  }
}
