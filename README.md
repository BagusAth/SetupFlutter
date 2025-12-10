# SetupFlutter

Searched codebase for "#codebase buatkan aku langkah-langkah detail dari awal setup hingga proyek ini jadi.", 17 results

# 📘 Panduan Lengkap: Membangun Aplikasi Buku Kontak dari Nol

Berikut adalah langkah-langkah **detail dan lengkap** untuk membuat project Aplikasi Buku Kontak ini dari awal hingga selesai.

---

## 📋 Daftar Isi

1. Persiapan Lingkungan
2. Membuat Project Flutter
3. Setup Firebase
4. Install Dependencies
5. Konfigurasi Android
6. Membuat Struktur Folder
7. Implementasi Kode
8. Menjalankan Aplikasi

---

## 1. Persiapan Lingkungan

### A. Install Flutter SDK
1. Download Flutter SDK dari [flutter.dev](https://docs.flutter.dev/get-started/install)
2. Extract ke folder (misal: `C:\flutter`)
3. Tambahkan ke PATH environment variable:
   ```
   C:\flutter\bin
   ```
4. Verifikasi instalasi:
   ```bash
   flutter doctor
   ```

### B. Install Tools Pendukung
- **VS Code**: [Download](https://code.visualstudio.com/)
  - Install extension: **Flutter** dan **Dart**
- **Android Studio**: [Download](https://developer.android.com/studio)
  - Untuk Android SDK dan Emulator
- **Node.js**: [Download](https://nodejs.org/)
  - Diperlukan untuk Firebase CLI

### C. Setup Android Emulator
1. Buka Android Studio → **Virtual Device Manager**
2. Create Device → Pilih **Pixel 6** atau device lain
3. Download system image (misal: **API 34**)
4. Finish dan jalankan emulator

---

## 2. Membuat Project Flutter

Buka terminal/PowerShell dan jalankan:

```bash
# 1. Buat project baru
flutter create contact_app

# 2. Masuk ke folder project
cd contact_app

# 3. Buka di VS Code
code .
```

**Hasil:** Folder project dengan struktur default Flutter.

---

## 3. Setup Firebase

### A. Install Firebase CLI

```bash
npm install -g firebase-tools
```

### B. Login ke Firebase

```bash
firebase login
```
Akan membuka browser untuk login ke akun Google Anda.

### C. Install FlutterFire CLI

```bash
dart pub global activate flutterfire_cli
```

### D. Konfigurasi Firebase di Project

```bash
flutterfire configure
```

**Langkah-langkah:**
1. Pilih **Create a new project** atau pilih project yang sudah ada
2. Masukkan nama project (misal: `my-contact-app`)
3. Pilih platform: **android** (centang dengan spasi, lalu Enter)
4. Tunggu proses selesai

**Hasil:** File firebase_options.dart otomatis dibuat.

### E. Setup Firestore Database (Di Browser)

1. Buka [Firebase Console](https://console.firebase.google.com/)
2. Pilih project yang baru dibuat
3. Menu sidebar: **Build** → **Firestore Database**
4. Klik **Create Database**
5. Pilih lokasi: `asia-southeast2` (Jakarta/Singapore)
6. Pilih **Start in Test Mode** → **Enable**

**Hasil:** Database Firestore siap digunakan.

---

## 4. Install Dependencies

Jalankan perintah ini di terminal:

```bash
flutter pub add firebase_core cloud_firestore image_picker
```

**Penjelasan:**
| Package | Fungsi |
|---------|--------|
| `firebase_core` | Koneksi utama ke Firebase |
| `cloud_firestore` | Database Cloud Firestore |
| `image_picker` | Akses kamera & galeri untuk foto profil |

**Hasil:** File pubspec.yaml otomatis terupdate:

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  firebase_core: ^3.6.0
  cloud_firestore: ^5.4.4
  image_picker: ^1.0.7
```

---

## 5. Konfigurasi Android

### A. Tambah Permission

Buka file AndroidManifest.xml dan tambahkan permission di atas tag `<application>`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Tambahkan permission ini -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="32"/>
    
    <application
        ...
```

### B. Cek minSdkVersion (Opsional)

Jika ada error terkait SDK version, buka build.gradle.kts dan pastikan:

```kotlin
defaultConfig {
    minSdk = 21  // atau lebih tinggi
    ...
}
```

---

## 6. Membuat Struktur Folder

Buat folder dan file berikut di dalam lib:

```
lib/
├── main.dart                      # Entry point (sudah ada)
├── firebase_options.dart          # Otomatis dari flutterfire
├── models/
│   └── contact.dart               # Model data kontak
├── screens/
│   ├── contact_list_screen.dart   # Halaman daftar kontak
│   ├── add_contact.dart           # Halaman tambah kontak
│   └── contact_detail.dart        # Halaman detail kontak
└── services/
    └── firebase_service.dart      # Service Firebase
```

**Cara buat folder di VS Code:**
- Klik kanan pada folder lib → **New Folder** → ketik `models`
- Ulangi untuk `screens` dan `services`

---

## 7. Implementasi Kode

### A. Model Contact (contact.dart)

```dart
class Contact {
  final String id;
  final String name;
  final String phone;
  final String email;
  final String address;
  final String notes;
  final bool isFavorite;
  final String? photoBase64;

  const Contact({
    this.id = '',
    required this.name,
    required this.phone,
    required this.email,
    this.address = '',
    this.notes = '',
    this.isFavorite = false,
    this.photoBase64,
  });

  // Membuat salinan objek dengan nilai baru
  Contact copyWith({
    String? id,
    String? name,
    String? phone,
    String? email,
    String? address,
    String? notes,
    bool? isFavorite,
    String? photoBase64,
  }) {
    return Contact(
      id: id ?? this.id,
      name: name ?? this.name,
      phone: phone ?? this.phone,
      email: email ?? this.email,
      address: address ?? this.address,
      notes: notes ?? this.notes,
      isFavorite: isFavorite ?? this.isFavorite,
      photoBase64: photoBase64 ?? this.photoBase64,
    );
  }

  // Mengambil inisial untuk avatar
  String getInitials() {
    if (name.trim().isEmpty) return 'NA';
    final parts = name.trim().split(RegExp(r'\s+'));
    if (parts.length >= 2) {
      return '${parts[0][0]}${parts[1][0]}'.toUpperCase();
    }
    return parts.first.substring(0, parts.first.length >= 2 ? 2 : 1).toUpperCase();
  }

  // Konversi ke Map untuk Firebase
  Map<String, dynamic> toMap() {
    return {
      'name': name,
      'phone': phone,
      'email': email,
      'address': address,
      'notes': notes,
      'isFavorite': isFavorite,
      'photoBase64': photoBase64,
    };
  }

  // Konversi dari Map Firebase ke Object
  factory Contact.fromMap(Map<String, dynamic>? data, {String id = ''}) {
    final map = data ?? <String, dynamic>{};
    return Contact(
      id: id,
      name: map['name'] ?? '',
      phone: map['phone'] ?? '',
      email: map['email'] ?? '',
      address: map['address'] ?? '',
      notes: map['notes'] ?? '',
      isFavorite: map['isFavorite'] ?? false,
      photoBase64: map['photoBase64'],
    );
  }
}
```

---

### B. Firebase Service (firebase_service.dart)

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/contact.dart';

class FirebaseService {
  final CollectionReference _contactsCollection =
      FirebaseFirestore.instance.collection('contacts');

  // CREATE - Tambah kontak baru
  Future<void> addContact(Contact contact) {
    return _contactsCollection.add(contact.toMap());
  }

  // READ - Ambil semua kontak (real-time)
  Stream<List<Contact>> getContacts() {
    return _contactsCollection
        .orderBy('name')
        .snapshots()
        .map((snapshot) {
      return snapshot.docs.map((doc) {
        return Contact.fromMap(
          doc.data() as Map<String, dynamic>,
          id: doc.id,
        );
      }).toList();
    });
  }

  // UPDATE - Edit kontak
  Future<void> updateContact(Contact contact) {
    return _contactsCollection.doc(contact.id).update(contact.toMap());
  }

  // DELETE - Hapus kontak
  Future<void> deleteContact(String id) {
    return _contactsCollection.doc(id).delete();
  }

  // Toggle Favorite
  Future<void> toggleFavorite(Contact contact) {
    return _contactsCollection.doc(contact.id).update({
      'isFavorite': !contact.isFavorite,
    });
  }
}
```

---

### C. Main Entry Point (main.dart)

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';
import 'screens/contact_list_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Buku Kontak',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF22D3EE),
        ),
        useMaterial3: true,
      ),
      home: const ContactListScreen(),
    );
  }
}
```

---

### D. Halaman Daftar Kontak (contact_list_screen.dart)

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import '../models/contact.dart';
import '../services/firebase_service.dart';
import 'add_contact.dart';
import 'contact_detail.dart';

class ContactListScreen extends StatefulWidget {
  const ContactListScreen({super.key});

  @override
  State<ContactListScreen> createState() => _ContactListScreenState();
}

class _ContactListScreenState extends State<ContactListScreen> {
  final FirebaseService _firebaseService = FirebaseService();
  final TextEditingController _searchController = TextEditingController();
  final GlobalKey _alphabetKey = GlobalKey();
  
  List<Contact> _allContacts = [];
  List<Contact> _filteredContacts = [];
  String? _popupLetter;
  final List<String> _alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('');

  @override
  void initState() {
    super.initState();
    _searchController.addListener(_filterContacts);
  }

  @override
  void dispose() {
    _searchController.dispose();
    super.dispose();
  }

  void _filterContacts() {
    final query = _searchController.text.toLowerCase();
    setState(() {
      _filteredContacts = _allContacts.where((contact) {
        final name = contact.name.toLowerCase();
        final phone = contact.phone.toLowerCase();
        return name.contains(query) || phone.contains(query);
      }).toList();
    });
  }

  Map<String, List<Contact>> _groupContacts(List<Contact> contacts) {
    final Map<String, List<Contact>> grouped = {};
    for (var contact in contacts) {
      final letter = contact.name.isNotEmpty 
          ? contact.name[0].toUpperCase() 
          : '#';
      grouped.putIfAbsent(letter, () => []);
      grouped[letter]!.add(contact);
    }
    return Map.fromEntries(
      grouped.entries.toList()..sort((a, b) => a.key.compareTo(b.key))
    );
  }

  void _scrollToLetter(String letter) {
    // Implementasi scroll ke huruf tertentu
    // Menggunakan ScrollController atau package tambahan
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: SafeArea(
        child: Column(
          children: [
            // Header
            _buildHeader(),
            // Search Bar
            _buildSearchBar(),
            // Contact List
            Expanded(
              child: Stack(
                children: [
                  _buildContactList(),
                  _buildAlphabetNavigator(),
                  if (_popupLetter != null) _buildPopupLetter(),
                ],
              ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Navigator.push(
            context,
            MaterialPageRoute(builder: (context) => const AddContactScreen()),
          );
        },
        backgroundColor: const Color(0xFF22D3EE),
        child: const Icon(Icons.add, color: Colors.white),
      ),
    );
  }

  Widget _buildHeader() {
    return Padding(
      padding: const EdgeInsets.all(20),
      child: Row(
        children: [
          Container(
            width: 56,
            height: 56,
            decoration: BoxDecoration(
              color: const Color(0xFF22D3EE).withOpacity(0.1),
              borderRadius: BorderRadius.circular(16),
            ),
            child: const Icon(
              Icons.contacts,
              color: Color(0xFF22D3EE),
              size: 28,
            ),
          ),
          const SizedBox(width: 16),
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text(
                'Daftar Kontak',
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                  color: Color(0xFF0F172A),
                ),
              ),
              Text(
                'Kelola dan temukan kontak dengan mudah',
                style: TextStyle(
                  fontSize: 14,
                  color: Colors.grey[600],
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }

  Widget _buildSearchBar() {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 20),
      child: TextField(
        controller: _searchController,
        decoration: InputDecoration(
          hintText: 'Cari kontak...',
          prefixIcon: const Icon(Icons.search, color: Color(0xFF94A3B8)),
          filled: true,
          fillColor: Colors.white,
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(16),
            borderSide: BorderSide.none,
          ),
          contentPadding: const EdgeInsets.symmetric(vertical: 16),
        ),
      ),
    );
  }

  Widget _buildContactList() {
    return StreamBuilder<List<Contact>>(
      stream: _firebaseService.getContacts(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }

        if (snapshot.hasError) {
          return Center(child: Text('Error: ${snapshot.error}'));
        }

        _allContacts = snapshot.data ?? [];
        final contacts = _searchController.text.isEmpty 
            ? _allContacts 
            : _filteredContacts;
        final groupedContacts = _groupContacts(contacts);

        if (contacts.isEmpty) {
          return const Center(
            child: Text('Tidak ada kontak', style: TextStyle(color: Colors.grey)),
          );
        }

        return ListView.builder(
          padding: const EdgeInsets.fromLTRB(20, 16, 60, 100),
          itemCount: groupedContacts.length,
          itemBuilder: (context, index) {
            final letter = groupedContacts.keys.elementAt(index);
            final contactsInGroup = groupedContacts[letter]!;

            return Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Section Header (Huruf)
                Padding(
                  padding: const EdgeInsets.only(left: 8, top: 16, bottom: 8),
                  child: Text(
                    letter,
                    style: const TextStyle(
                      fontSize: 14,
                      fontWeight: FontWeight.bold,
                      color: Color(0xFF94A3B8),
                    ),
                  ),
                ),
                // Contact Cards
                ...contactsInGroup.map((contact) => _buildContactCard(contact)),
              ],
            );
          },
        );
      },
    );
  }

  Widget _buildContactCard(Contact contact) {
    return GestureDetector(
      onTap: () {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => ContactDetailScreen(contact: contact),
          ),
        );
      },
      child: Container(
        margin: const EdgeInsets.only(bottom: 12),
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(16),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.04),
              blurRadius: 8,
              offset: const Offset(0, 2),
            ),
          ],
        ),
        child: Row(
          children: [
            // Avatar
            Container(
              width: 48,
              height: 48,
              decoration: BoxDecoration(
                color: const Color(0xFF22D3EE).withOpacity(0.1),
                shape: BoxShape.circle,
                image: contact.photoBase64 != null
                    ? DecorationImage(
                        image: MemoryImage(base64Decode(contact.photoBase64!)),
                        fit: BoxFit.cover,
                      )
                    : null,
              ),
              child: contact.photoBase64 == null
                  ? Center(
                      child: Text(
                        contact.getInitials(),
                        style: const TextStyle(
                          color: Color(0xFF22D3EE),
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    )
                  : null,
            ),
            const SizedBox(width: 16),
            // Info
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    contact.name,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.w600,
                      color: Color(0xFF0F172A),
                    ),
                  ),
                  const SizedBox(height: 4),
                  Row(
                    children: [
                      const Icon(Icons.phone, size: 14, color: Color(0xFF94A3B8)),
                      const SizedBox(width: 4),
                      Text(
                        contact.phone,
                        style: const TextStyle(
                          fontSize: 13,
                          color: Color(0xFF64748B),
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
            // Favorite Icon
            if (contact.isFavorite)
              const Icon(Icons.star, color: Color(0xFFFBBF24), size: 20),
          ],
        ),
      ),
    );
  }

  Widget _buildAlphabetNavigator() {
    return Positioned(
      right: 8,
      top: 20,
      bottom: 20,
      child: GestureDetector(
        key: _alphabetKey,
        onVerticalDragStart: (details) => _handleDrag(details.localPosition),
        onVerticalDragUpdate: (details) => _handleDrag(details.localPosition),
        onVerticalDragEnd: (_) => setState(() => _popupLetter = null),
        child: Container(
          width: 40,
          padding: const EdgeInsets.symmetric(vertical: 12),
          decoration: BoxDecoration(
            color: Colors.white.withOpacity(0.8),
            borderRadius: BorderRadius.circular(20),
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: _alphabet.map((letter) {
              final hasContacts = _allContacts.any(
                (c) => c.name.toUpperCase().startsWith(letter),
              );
              return Text(
                letter,
                style: TextStyle(
                  fontSize: 11,
                  fontWeight: hasContacts ? FontWeight.bold : FontWeight.normal,
                  color: hasContacts 
                      ? const Color(0xFF22D3EE) 
                      : const Color(0xFFCBD5E1),
                ),
              );
            }).toList(),
          ),
        ),
      ),
    );
  }

  void _handleDrag(Offset position) {
    final itemHeight = 20.0;
    final index = (position.dy / itemHeight).floor().clamp(0, _alphabet.length - 1);
    final letter = _alphabet[index];
    
    setState(() {
      _popupLetter = letter;
    });
    _scrollToLetter(letter);
  }

  Widget _buildPopupLetter() {
    return Center(
      child: Container(
        width: 100,
        height: 100,
        decoration: BoxDecoration(
          color: const Color(0xFF22D3EE),
          borderRadius: BorderRadius.circular(16),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.3),
              blurRadius: 20,
            ),
          ],
        ),
        child: Center(
          child: Text(
            _popupLetter!,
            style: const TextStyle(
              fontSize: 48,
              fontWeight: FontWeight.bold,
              color: Colors.white,
            ),
          ),
        ),
      ),
    );
  }
}
```

---

### E. Halaman Tambah Kontak (add_contact.dart)

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import '../models/contact.dart';
import '../services/firebase_service.dart';

class AddContactScreen extends StatefulWidget {
  const AddContactScreen({super.key});

  @override
  State<AddContactScreen> createState() => _AddContactScreenState();
}

class _AddContactScreenState extends State<AddContactScreen> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _phoneController = TextEditingController();
  final _emailController = TextEditingController();
  final _addressController = TextEditingController();
  final _notesController = TextEditingController();
  final _firebaseService = FirebaseService();
  final _imagePicker = ImagePicker();
  
  bool _isLoading = false;
  String? _photoBase64;

  @override
  void dispose() {
    _nameController.dispose();
    _phoneController.dispose();
    _emailController.dispose();
    _addressController.dispose();
    _notesController.dispose();
    super.dispose();
  }

  Future<void> _pickImage() async {
    final XFile? image = await _imagePicker.pickImage(
      source: ImageSource.gallery,
      maxWidth: 512,
      maxHeight: 512,
      imageQuality: 85,
    );
    if (image != null) {
      final bytes = await image.readAsBytes();
      setState(() {
        _photoBase64 = base64Encode(bytes);
      });
    }
  }

  Future<void> _takePhoto() async {
    final XFile? image = await _imagePicker.pickImage(
      source: ImageSource.camera,
      maxWidth: 512,
      maxHeight: 512,
      imageQuality: 85,
    );
    if (image != null) {
      final bytes = await image.readAsBytes();
      setState(() {
        _photoBase64 = base64Encode(bytes);
      });
    }
  }

  void _showPhotoOptions() {
    showModalBottomSheet(
      context: context,
      builder: (context) => SafeArea(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            ListTile(
              leading: const Icon(Icons.photo_library),
              title: const Text('Pilih dari Galeri'),
              onTap: () {
                Navigator.pop(context);
                _pickImage();
              },
            ),
            ListTile(
              leading: const Icon(Icons.camera_alt),
              title: const Text('Ambil Foto'),
              onTap: () {
                Navigator.pop(context);
                _takePhoto();
              },
            ),
            if (_photoBase64 != null)
              ListTile(
                leading: const Icon(Icons.delete, color: Colors.red),
                title: const Text('Hapus Foto', style: TextStyle(color: Colors.red)),
                onTap: () {
                  Navigator.pop(context);
                  setState(() => _photoBase64 = null);
                },
              ),
          ],
        ),
      ),
    );
  }

  Future<void> _saveContact() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() => _isLoading = true);

    try {
      final contact = Contact(
        name: _nameController.text.trim(),
        phone: _phoneController.text.trim(),
        email: _emailController.text.trim(),
        address: _addressController.text.trim(),
        notes: _notesController.text.trim(),
        photoBase64: _photoBase64,
      );

      await _firebaseService.addContact(contact);
      
      if (mounted) {
        Navigator.pop(context);
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Kontak berhasil disimpan')),
        );
      }
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Error: $e')),
      );
    } finally {
      setState(() => _isLoading = false);
    }
  }

  String _getInitials() {
    final name = _nameController.text.trim();
    if (name.isEmpty) return 'NA';
    final parts = name.split(RegExp(r'\s+'));
    if (parts.length >= 2) {
      return '${parts[0][0]}${parts[1][0]}'.toUpperCase();
    }
    return name.substring(0, name.length >= 2 ? 2 : 1).toUpperCase();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: SafeArea(
        child: Column(
          children: [
            // Header
            Padding(
              padding: const EdgeInsets.all(16),
              child: Row(
                children: [
                  IconButton(
                    onPressed: () => Navigator.pop(context),
                    icon: const Icon(Icons.arrow_back),
                  ),
                  const Expanded(
                    child: Text(
                      'Tambah Kontak',
                      textAlign: TextAlign.center,
                      style: TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                  const SizedBox(width: 48),
                ],
              ),
            ),
            // Form
            Expanded(
              child: SingleChildScrollView(
                child: Form(
                  key: _formKey,
                  child: Column(
                    children: [
                      // Avatar Section
                      Container(
                        width: double.infinity,
                        padding: const EdgeInsets.all(32),
                        color: const Color(0xFFEFFCFD),
                        child: Column(
                          children: [
                            GestureDetector(
                              onTap: _showPhotoOptions,
                              child: Container(
                                width: 112,
                                height: 112,
                                decoration: BoxDecoration(
                                  color: const Color(0xFF22D3EE).withOpacity(0.1),
                                  shape: BoxShape.circle,
                                  border: Border.all(color: Colors.white, width: 4),
                                  image: _photoBase64 != null
                                      ? DecorationImage(
                                          image: MemoryImage(base64Decode(_photoBase64!)),
                                          fit: BoxFit.cover,
                                        )
                                      : null,
                                ),
                                child: _photoBase64 == null
                                    ? Center(
                                        child: Text(
                                          _getInitials(),
                                          style: const TextStyle(
                                            fontSize: 36,
                                            fontWeight: FontWeight.bold,
                                            color: Color(0xFF22D3EE),
                                          ),
                                        ),
                                      )
                                    : null,
                              ),
                            ),
                            const SizedBox(height: 16),
                            TextButton.icon(
                              onPressed: _showPhotoOptions,
                              icon: const Icon(Icons.camera_alt),
                              label: Text(_photoBase64 == null ? 'Tambah Foto' : 'Ganti Foto'),
                            ),
                          ],
                        ),
                      ),
                      // Input Fields
                      Container(
                        margin: const EdgeInsets.all(20),
                        padding: const EdgeInsets.all(24),
                        decoration: BoxDecoration(
                          color: Colors.white,
                          borderRadius: BorderRadius.circular(24),
                          boxShadow: [
                            BoxShadow(
                              color: Colors.black.withOpacity(0.04),
                              blurRadius: 8,
                            ),
                          ],
                        ),
                        child: Column(
                          children: [
                            _buildTextField(
                              controller: _nameController,
                              label: 'Nama',
                              icon: Icons.person,
                              validator: (v) => v!.isEmpty ? 'Nama wajib diisi' : null,
                            ),
                            _buildTextField(
                              controller: _phoneController,
                              label: 'Nomor Telepon',
                              icon: Icons.phone,
                              keyboardType: TextInputType.phone,
                              validator: (v) => v!.isEmpty ? 'Telepon wajib diisi' : null,
                            ),
                            _buildTextField(
                              controller: _emailController,
                              label: 'Email',
                              icon: Icons.email,
                              keyboardType: TextInputType.emailAddress,
                            ),
                            _buildTextField(
                              controller: _addressController,
                              label: 'Alamat',
                              icon: Icons.location_on,
                            ),
                            _buildTextField(
                              controller: _notesController,
                              label: 'Catatan',
                              icon: Icons.note,
                              maxLines: 3,
                            ),
                          ],
                        ),
                      ),
                      // Save Button
                      Padding(
                        padding: const EdgeInsets.fromLTRB(20, 0, 20, 32),
                        child: SizedBox(
                          width: double.infinity,
                          child: ElevatedButton(
                            onPressed: _isLoading ? null : _saveContact,
                            style: ElevatedButton.styleFrom(
                              backgroundColor: const Color(0xFF22D3EE),
                              foregroundColor: Colors.white,
                              padding: const EdgeInsets.symmetric(vertical: 16),
                              shape: RoundedRectangleBorder(
                                borderRadius: BorderRadius.circular(16),
                              ),
                            ),
                            child: _isLoading
                                ? const SizedBox(
                                    width: 24,
                                    height: 24,
                                    child: CircularProgressIndicator(
                                      color: Colors.white,
                                      strokeWidth: 2,
                                    ),
                                  )
                                : const Text(
                                    'Simpan Kontak',
                                    style: TextStyle(
                                      fontSize: 16,
                                      fontWeight: FontWeight.bold,
                                    ),
                                  ),
                          ),
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildTextField({
    required TextEditingController controller,
    required String label,
    required IconData icon,
    TextInputType? keyboardType,
    String? Function(String?)? validator,
    int maxLines = 1,
  }) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 16),
      child: TextFormField(
        controller: controller,
        keyboardType: keyboardType,
        validator: validator,
        maxLines: maxLines,
        decoration: InputDecoration(
          labelText: label,
          prefixIcon: Icon(icon, color: const Color(0xFF94A3B8)),
          filled: true,
          fillColor: const Color(0xFFF8FAFC),
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(14),
            borderSide: BorderSide.none,
          ),
          focusedBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(14),
            borderSide: const BorderSide(color: Color(0xFF22D3EE), width: 2),
          ),
        ),
      ),
    );
  }
}
```

---

### F. Halaman Detail Kontak (contact_detail.dart)

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import '../models/contact.dart';
import '../services/firebase_service.dart';

class ContactDetailScreen extends StatefulWidget {
  final Contact? contact;
  
  const ContactDetailScreen({super.key, this.contact});

  @override
  State<ContactDetailScreen> createState() => _ContactDetailScreenState();
}

class _ContactDetailScreenState extends State<ContactDetailScreen> {
  final FirebaseService _firebaseService = FirebaseService();
  late Contact _contact;
  
  late TextEditingController _nameController;
  late TextEditingController _phoneController;
  late TextEditingController _emailController;
  late TextEditingController _addressController;
  late TextEditingController _notesController;

  bool _isEditing = false;
  bool _isFavorite = false;

  @override
  void initState() {
    super.initState();
    _contact = widget.contact ?? Contact(name: '', phone: '', email: '');
    _isFavorite = _contact.isFavorite;
    
    _nameController = TextEditingController(text: _contact.name);
    _phoneController = TextEditingController(text: _contact.phone);
    _emailController = TextEditingController(text: _contact.email);
    _addressController = TextEditingController(text: _contact.address);
    _notesController = TextEditingController(text: _contact.notes);
  }

  @override
  void dispose() {
    _nameController.dispose();
    _phoneController.dispose();
    _emailController.dispose();
    _addressController.dispose();
    _notesController.dispose();
    super.dispose();
  }

  Future<void> _saveChanges() async {
    final updatedContact = _contact.copyWith(
      name: _nameController.text.trim(),
      phone: _phoneController.text.trim(),
      email: _emailController.text.trim(),
      address: _addressController.text.trim(),
      notes: _notesController.text.trim(),
      isFavorite: _isFavorite,
    );

    await _firebaseService.updateContact(updatedContact);
    setState(() {
      _contact = updatedContact;
      _isEditing = false;
    });

    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Kontak berhasil diperbarui')),
      );
    }
  }

  Future<void> _deleteContact() async {
    final confirm = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Hapus Kontak'),
        content: Text('Yakin ingin menghapus ${_contact.name}?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Batal'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Hapus', style: TextStyle(color: Colors.red)),
          ),
        ],
      ),
    );

    if (confirm == true) {
      await _firebaseService.deleteContact(_contact.id);
      if (mounted) {
        Navigator.pop(context);
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Kontak berhasil dihapus')),
        );
      }
    }
  }

  Future<void> _toggleFavorite() async {
    setState(() => _isFavorite = !_isFavorite);
    await _firebaseService.toggleFavorite(_contact);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF8FAFC),
      body: SafeArea(
        child: Column(
          children: [
            // Header
            Padding(
              padding: const EdgeInsets.all(16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  IconButton(
                    onPressed: () => Navigator.pop(context),
                    icon: const Icon(Icons.arrow_back),
                  ),
                  Row(
                    children: [
                      if (!_isEditing) ...[
                        IconButton(
                          onPressed: () => setState(() => _isEditing = true),
                          icon: const Icon(Icons.edit),
                        ),
                        IconButton(
                          onPressed: _deleteContact,
                          icon: const Icon(Icons.delete, color: Colors.red),
                        ),
                      ] else ...[
                        TextButton(
                          onPressed: () => setState(() => _isEditing = false),
                          child: const Text('Batal'),
                        ),
                        TextButton(
                          onPressed: _saveChanges,
                          child: const Text('Simpan'),
                        ),
                      ],
                    ],
                  ),
                ],
              ),
            ),
            // Content
            Expanded(
              child: SingleChildScrollView(
                child: Column(
                  children: [
                    // Avatar Section
                    Container(
                      width: double.infinity,
                      padding: const EdgeInsets.all(32),
                      color: const Color(0xFFEFFCFD),
                      child: Column(
                        children: [
                          Container(
                            width: 112,
                            height: 112,
                            decoration: BoxDecoration(
                              color: const Color(0xFF22D3EE).withOpacity(0.1),
                              shape: BoxShape.circle,
                              border: Border.all(color: Colors.white, width: 4),
                              image: _contact.photoBase64 != null
                                  ? DecorationImage(
                                      image: MemoryImage(base64Decode(_contact.photoBase64!)),
                                      fit: BoxFit.cover,
                                    )
                                  : null,
                            ),
                            child: _contact.photoBase64 == null
                                ? Center(
                                    child: Text(
                                      _contact.getInitials(),
                                      style: const TextStyle(
                                        fontSize: 36,
                                        fontWeight: FontWeight.bold,
                                        color: Color(0xFF22D3EE),
                                      ),
                                    ),
                                  )
                                : null,
                          ),
                          const SizedBox(height: 16),
                          Text(
                            _contact.name,
                            style: const TextStyle(
                              fontSize: 24,
                              fontWeight: FontWeight.bold,
                              color: Color(0xFF0F172A),
                            ),
                          ),
                        ],
                      ),
                    ),
                    // Quick Actions (hanya tampil saat tidak edit)
                    if (!_isEditing) ...[
                      Padding(
                        padding: const EdgeInsets.all(20),
                        child: Row(
                          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                          children: [
                            _buildQuickAction(Icons.phone, 'Telepon'),
                            _buildQuickAction(Icons.message, 'Pesan'),
                            _buildQuickAction(
                              _isFavorite ? Icons.star : Icons.star_border,
                              'Favorit',
                              onTap: _toggleFavorite,
                            ),
                          ],
                        ),
                      ),
                    ],
                    // Info Fields
                    Container(
                      margin: const EdgeInsets.all(20),
                      padding: const EdgeInsets.all(24),
                      decoration: BoxDecoration(
                        color: Colors.white,
                        borderRadius: BorderRadius.circular(24),
                        boxShadow: [
                          BoxShadow(
                            color: Colors.black.withOpacity(0.04),
                            blurRadius: 8,
                          ),
                        ],
                      ),
                      child: Column(
                        children: [
                          _buildInfoField('Nama', _nameController, Icons.person),
                          _buildInfoField('Telepon', _phoneController, Icons.phone),
                          _buildInfoField('Email', _emailController, Icons.email),
                          _buildInfoField('Alamat', _addressController, Icons.location_on),
                          _buildInfoField('Catatan', _notesController, Icons.note),
                        ],
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildQuickAction(IconData icon, String label, {VoidCallback? onTap}) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(16),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.04),
              blurRadius: 8,
            ),
          ],
        ),
        child: Column(
          children: [
            Icon(icon, color: const Color(0xFF64748B)),
            const SizedBox(height: 8),
            Text(label, style: const TextStyle(color: Color(0xFF64748B))),
          ],
        ),
      ),
    );
  }

  Widget _buildInfoField(String label, TextEditingController controller, IconData icon) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            label,
            style: const TextStyle(
              fontSize: 13,
              color: Color(0xFF94A3B8),
            ),
          ),
          const SizedBox(height: 8),
          TextFormField(
            controller: controller,
            readOnly: !_isEditing,
            decoration: InputDecoration(
              prefixIcon: Icon(icon, color: const Color(0xFF94A3B8)),
              filled: true,
              fillColor: const Color(0xFFF8FAFC),
              border: OutlineInputBorder(
                borderRadius: BorderRadius.circular(14),
                borderSide: BorderSide.none,
              ),
              focusedBorder: OutlineInputBorder(
                borderRadius: BorderRadius.circular(14),
                borderSide: const BorderSide(color: Color(0xFF22D3EE), width: 2),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 8. Menjalankan Aplikasi

### A. Pastikan Emulator Menyala
Buka Android Studio → Virtual Device Manager → Jalankan emulator

### B. Jalankan Aplikasi

```bash
flutter run
```

### C. Troubleshooting

**Error `minSdkVersion`:**
```kotlin
// android/app/build.gradle.kts
defaultConfig {
    minSdk = 21
    ...
}
```

**Error `MissingPluginException` (image_picker):**
```bash
flutter clean
flutter pub get
flutter run
```

---

## ✅ Hasil Akhir

Setelah mengikuti semua langkah di atas, Anda akan memiliki:

| Fitur | Deskripsi |
|-------|-----------|
| 📋 **Daftar Kontak** | List dengan alphabet navigator |
| ➕ **Tambah Kontak** | Form input dengan upload foto |
| 👤 **Detail Kontak** | Lihat info lengkap + edit/hapus |
| 📷 **Foto Profil** | Upload dari galeri/kamera (Base64) |
| 🔍 **Pencarian** | Filter kontak real-time |
| ⭐ **Favorit** | Tandai kontak penting |
| ☁️ **Cloud Sync** | Data tersimpan di Firebase |

---

## 📚 Referensi

- [Flutter Documentation](https://docs.flutter.dev/)
- [Firebase for Flutter](https://firebase.google.com/docs/flutter/setup)
- [Cloud Firestore](https://firebase.google.com/docs/firestore)
- [Image Picker Package](https://pub.dev/packages/image_picker)
