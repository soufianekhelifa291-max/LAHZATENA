# LAHZATENAlib/
├── main.dart
├── screens/
│   ├── home_screen.dart
│   └── viewer_screen.dart
└── services/
    └── gallery_service.dart

================== lib/main.dart ==================
import 'package:flutter/material.dart';
import 'screens/home_screen.dart';

void main() {
  runApp(LahazatonaApp());
}

class LahazatonaApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'لحظاتنا',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        fontFamily: 'Cairo',
      ),
      home: HomeScreen(),
    );
  }
}

================== lib/services/gallery_service.dart ==================
import 'package:photo_manager/photo_manager.dart';

class GalleryService {
  Future<List<AssetEntity>> getImages() async {
    final permission = await PhotoManager.requestPermissionExtend();

    if (permission.isAuth) {
      final albums = await PhotoManager.getAssetPathList(
        type: RequestType.image,
      );

      final recentAlbum = albums.first;

      final images = await recentAlbum.getAssetListPaged(
        page: 0,
        size: 200,
      );

      return images;
    }
    return [];
  }
}

================== lib/screens/home_screen.dart ==================
import 'package:flutter/material.dart';
import 'package:photo_manager/photo_manager.dart';
import '../services/gallery_service.dart';
import 'viewer_screen.dart';
import 'package:animations/animations.dart';

class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  List<AssetEntity> images = [];
  List<AssetEntity> filtered = [];

  @override
  void initState() {
    super.initState();
    loadImages();
  }

  void loadImages() async {
    final data = await GalleryService().getImages();
    setState(() {
      images = data;
      filtered = data;
    });
  }

  void search(String query) {
    setState(() {
      filtered = images.where((img) {
        return img.title?.toLowerCase().contains(query.toLowerCase()) ?? false;
      }).toList();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("لحظاتنا"),
        backgroundColor: Color(0xFFF1A9E1),
      ),
      body: Container(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [
              Color(0xFFF1A9E1),
              Color(0xFF87CEEB),
            ],
          ),
        ),
        child: Column(
          children: [
            // 🔍 Search
            Padding(
              padding: EdgeInsets.all(10),
              child: TextField(
                onChanged: search,
                decoration: InputDecoration(
                  hintText: "ابحث عن صورة...",
                  filled: true,
                  fillColor: Colors.white,
                  prefixIcon: Icon(Icons.search),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(20),
                  ),
                ),
              ),
            ),

            // 📸 Grid
            Expanded(
              child: GridView.builder(
                itemCount: filtered.length,
                gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 3,
                ),
                itemBuilder: (context, index) {
                  return FutureBuilder(
                    future: filtered[index].thumbnailData,
                    builder: (_, snapshot) {
                      if (!snapshot.hasData) return Container();

                      return OpenContainer(
                        closedElevation: 0,
                        openBuilder: (_, __) =>
                            ViewerScreen(image: filtered[index]),
                        closedBuilder: (_, open) => GestureDetector(
                          onTap: open,
                          child: Hero(
                            tag: filtered[index].id,
                            child: ClipRRect(
                              borderRadius: BorderRadius.circular(15),
                              child: Image.memory(
                                snapshot.data!,
                                fit: BoxFit.cover,
                              ),
                            ),
                          ),
                        ),
                      );
                    },
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

================== lib/screens/viewer_screen.dart ==================
import 'package:flutter/material.dart';
import 'package:photo_manager/photo_manager.dart';

class ViewerScreen extends StatelessWidget {
  final AssetEntity image;

  ViewerScreen({required this.image});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: FutureBuilder(
        future: image.originBytes,
        builder: (_, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }

          return Center(
            child: Hero(
              tag: image.id,
              child: InteractiveViewer(
                child: Image.memory(snapshot.data!),
              ),
            ),
          );
        },
      ),
    );
  }
}

================== pubspec.yaml ==================
dependencies:
  flutter:
    sdk: flutter
  photo_manager: ^3.0.0
  permission_handler: ^11.0.0
  animations: ^2.0.7

================== android/app/src/main/AndroidManifest.xml ==================
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
