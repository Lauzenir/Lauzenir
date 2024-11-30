import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_database/firebase_database.dart';
import 'package:barcode_scan2/barcode_scan2.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicializa o Firebase
  runApp(PharmaControlApp());
}

class PharmaControlApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'PharmaControl',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: HomeScreen(),
    );
  }
}

class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  String scannedCode = '';
  Map<String, dynamic>? productDetails;

  // Referência ao banco de dados Firebase
  final DatabaseReference dbRef = FirebaseDatabase.instance.ref().child('products');

  // Função para escanear código de barras
  Future<void> scanBarcode() async {
    try {
      var result = await BarcodeScanner.scan();
      setState(() {
        scannedCode = result.rawContent;
      });
      if (scannedCode.isNotEmpty) {
        fetchProductDetails(scannedCode);
      }
    } catch (e) {
      setState(() {
        scannedCode = 'Erro ao escanear código!';
      });
    }
  }

  // Função para buscar detalhes do produto no Firebase
  Future<void> fetchProductDetails(String code) async {
    final snapshot = await dbRef.child(code).get();
    if (snapshot.exists) {
      setState(() {
        productDetails = Map<String, dynamic>.from(snapshot.value as Map);
      });
    } else {
      setState(() {
        productDetails = null;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('PharmaControl'),
        actions: [
          IconButton(
            icon: Icon(Icons.qr_code_scanner),
            onPressed: scanBarcode,
          ),
        ],
      ),
      body: Center(
        child: productDetails == null
            ? Text(
                scannedCode.isNotEmpty
                    ? 'Produto não encontrado no banco de dados.'
                    : 'Escaneie um código de barras para começar.',
                style: TextStyle(fontSize: 18),
                textAlign: TextAlign.center,
              )
            : ProductDetailScreen(details: productDetails!),
      ),
    );
  }
}

class ProductDetailScreen extends StatelessWidget {
  final Map<String, dynamic> details;

  ProductDetailScreen({required this.details});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('Nome: ${details['name']}', style: TextStyle(fontSize: 18)),
          Text('Preço de Venda: R\$ ${details['price_sale']}', style: TextStyle(fontSize: 18)),
          Text('Preço de Custo: R\$ ${details['price_cost']}', style: TextStyle(fontSize: 18)),
          Text('Quantidade: ${details['quantity']}', style: TextStyle(fontSize: 18)),
          Text('Validade: ${details['expiry_date']}', style: TextStyle(fontSize: 18)),
        ],
      ),
    );
  }
}
