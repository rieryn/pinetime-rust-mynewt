# Auto Convert Go to Dart with an Abstract Syntax Tree

![Work-in-progress PineTime Companion App on iPhone, converted from Go to Flutter and Dart](https://lupyuen.github.io/images/bloc-ios.jpg)

_Work-in-progress PineTime Companion App on iPhone, converted from Go to Flutter and Dart_

TODO

Let's learn how this Go code...

[`nmxact/nmp/image.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/master/nmxact/nmp/image.go)

```go
//  Convert From Go...
//  Go Struct
type ImageUploadReq struct {
  NmpBase  `codec:"-"`
  ImageNum uint8  `codec:"image"`
  Off      uint32 `codec:"off"`
  Len      uint32 `codec:"len,omitempty"`
  DataSha  []byte `codec:"sha,omitempty"`
  Upgrade  bool   `codec:"upgrade,omitempty"`
  Data     []byte `codec:"data"`
}

//  Go Function
func NewImageUploadReq() *ImageUploadReq {
  r := &ImageUploadReq{}
  fillNmpReq(r, NMP_OP_WRITE, NMP_GROUP_IMAGE, NMP_ID_IMAGE_UPLOAD)
  return r
}
```

Was automatically converted to this Dart code...

[`dart/nmp/image.dart`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/nmp/image.dart)

```dart
//  Converted To Dart...
//  Converted Dart Class
class ImageUploadReq 
  with NmpBase       //  Get and set SMP Message Header
  implements NmpReq  //  SMP Request Message
{
  int ImageNum; //  image: uint8
  int Off;      //  off: uint32
  int Len;      //  len: uint32
  typed.Uint8Buffer DataSha;  //  sha: []byte
  bool Upgrade;               //  upgrade: bool
  typed.Uint8Buffer Data;     //  data: []byte

  NmpMsg Msg() { return MsgFromReq(this); }

  /// Encode the SMP Request fields to CBOR
  void Encode(cbor.MapBuilder builder) {
    builder.writeString("image");
    builder.writeInt(ImageNum); // uint8
    //  ...Omitted...
    builder.writeString("data");
    builder.writeArray(Data);   // []byte
  }
}

//  Converted Dart Function
ImageUploadReq NewImageUploadReq() {
  var r = ImageUploadReq();
  fillNmpReq(r, NMP_OP_WRITE, NMP_GROUP_IMAGE, NMP_ID_IMAGE_UPLOAD);
  return r;
}
```

# Why Auto Convert Go to Dart?

TODO

Both languages were created at Google, yet so different

Minimise errors in conversion

Huge volume of code

Consistency of conversion

Easy cross reference

# Manual Conversion Gets Tiring

TODO

Semicolons

Name Type -> Type Name

# Generate Abstract Syntax Tree

TODO

Here is the code that generates an Abstract Syntax Tree given a block of Go code: [`dart/convert.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/convert.go)

```go
// Inspect the Abstract Syntax Tree of our Go code and convert to Dart
func convertGoToDart() {
  fmt.Printf("//  Go Code...\n%s\n", src)
  fmt.Println("//  Converted To Dart...\n")

  // Create the Abstract Syntax Tree by parsing Go code
  fileset := token.NewFileSet()                            // Positions are relative to fileset
  node, err := parser.ParseFile(fileset, "src.go", src, 0) // Change src to nil to parse a file instead of string
  if err != nil {
    panic(err)
  }
  ...
}
```

Why use Abstract Syntax Tree not LLVM?

# Walk the Abstract Syntax Tree

TODO

Here is the code that walks an Abstract Syntax Tree and converts each chunk of Go code: [`dart/convert.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/convert.go)

```go
// Inspect the Abstract Syntax Tree of our Go code and convert to Dart
func convertGoToDart() {
  // Omitted: Create the Abstract Syntax Tree by parsing Go code
  ...
  // Convert all Go Struct and Function Declarations
  for _, decl := range node.Decls {
    // ast.Print(fileset, decl)
    switch decl := decl.(type) {
    case *ast.GenDecl:
      // Convert Go Struct to Dart
      convertStruct(fileset, decl)
    case *ast.FuncDecl:
      // Convert Go Function to Dart
      convertFunction(fileset, decl)
    default:
      fmt.Println("*** Unknown Decl:")
      ast.Print(fileset, decl)
    }
  }
}
```

# Auto Convert Go Type to Dart

TODO

Here is the code that converts a Go Type to Dart and CBOR: [`dart/convert.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/convert.go)

```go
// DartField represents a Go Struct Field converted to Dart and CBOR
type DartField struct {
  Name     string // "Len"
  CborName string // "len"
  GoType   string // "uint32"
  DartType string // "int"
  CborType string // "Int"
}
```

```go
// Convert a Go type to Dart type and CBOR type
func convertType(typeName string) (string, string) {
  switch typeName {
  case "bool":
    return "bool", "Bool"
  case "uint8":
    return "int", "Int"
  case "uint16":
    return "int", "Int"
  case "uint32":
    return "int", "Int"
  case "[]byte":
    return "typed.Uint8Buffer", "Array"
  default:
    return "UNKNOWN", "UNKNOWN"
  }
}
```


```go
// Convert a Go Struct Field to Dart
func convertField(fileset *token.FileSet, astField *ast.Field) DartField {
  dartField := DartField{}
  if len(astField.Names) > 0 {
    dartField.Name = astField.Names[0].Name // "Len"
  }
  dartField.GoType = fmt.Sprintf("%v", astField.Type) // "uint32"
  // Handle "&{181 <nil> byte}" as "[]byte"
  if strings.HasPrefix(dartField.GoType, "&{") && strings.HasSuffix(dartField.GoType, " byte}") {
    dartField.GoType = "[]byte"
  }
  dartField.DartType, dartField.CborType = convertType(dartField.GoType) // "int"

  // Convert a Field Tag like `codec:"len,omitempty"`. CborName will be set to "len".
  if astField.Tag != nil {
    dartField.CborName = strings.Split(astField.Tag.Value, ",")[0]
    dartField.CborName = strings.Replace(dartField.CborName, "codec:", "", 1)
    dartField.CborName = strings.Replace(dartField.CborName, `"`, "", 2)
    dartField.CborName = strings.Replace(dartField.CborName, "`", "", 2)
  }
  // fmt.Printf("field: %s,\tcbor: %s,\ttype: %s,\tdart: %s\n", dartField.Name, dartField.CborName, dartField.GoType, dartField.DartType)
  return dartField
}
```

# Auto Convert Go Struct to Dart

TODO

[`nmxact/nmp/image.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/master/nmxact/nmp/image.go)

```go
//  Convert From Go...
//  Go Struct
type ImageUploadReq struct {
  NmpBase  `codec:"-"`
  ImageNum uint8  `codec:"image"`
  Off      uint32 `codec:"off"`
  Len      uint32 `codec:"len,omitempty"`
  DataSha  []byte `codec:"sha,omitempty"`
  Upgrade  bool   `codec:"upgrade,omitempty"`
  Data     []byte `codec:"data"`
}
```

[`dart/nmp/image.dart`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/nmp/image.dart)

```dart
//  Converted To Dart...
//  Converted Dart Class
class ImageUploadReq 
  with NmpBase       //  Get and set SMP Message Header
  implements NmpReq  //  SMP Request Message
{
  int ImageNum; //  image: uint8
  int Off;      //  off: uint32
  int Len;      //  len: uint32
  typed.Uint8Buffer DataSha;  //  sha: []byte
  bool Upgrade;               //  upgrade: bool
  typed.Uint8Buffer Data;     //  data: []byte

  NmpMsg Msg() { return MsgFromReq(this); }

  /// Encode the SMP Request fields to CBOR
  void Encode(cbor.MapBuilder builder) {
    //  ...Omitted...
  }
}
```

Here is the code that converts a Go Struct to Dart: [`dart/convert.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/convert.go)

```go
// Convert Go Struct to Dart
func convertStruct(fileset *token.FileSet, decl *ast.GenDecl) {
  // ast.Print(fileset, decl)
  switch decl.Tok.String() {
  case "type":
    // Convert a type declaration
    for _, spec := range decl.Specs {
      // ast.Print(fileset, spec)
      switch spec := spec.(type) {
      case *ast.TypeSpec:
        // Get the struct name and output the Dart class
        typeName := spec.Name.Name // "NmpHdr"
        fmt.Printf("class %s ", typeName)

        // If this is a request message struct...
        if strings.HasSuffix(typeName, "Req") {
          // Add the Dart mixin and interface classes
          fmt.Println("")
          fmt.Println("  with NmpBase       //  Get and set SMP Message Header")
          fmt.Println("  implements NmpReq  //  SMP Request Message")
        }
        fmt.Println("{")

        switch structType := spec.Type.(type) {
        case *ast.StructType: // "struct {"
          // Convert the struct fields
          fields := structType.Fields.List
          convertFields(fileset, fields)
          fmt.Println("")

          // If this is a request message struct...
          if strings.HasSuffix(typeName, "Req") {
            fmt.Println("  NmpMsg Msg() { return MsgFromReq(this); }\n")
            // Generate CBOR encoder
            generateCborEncoder(fileset, fields)
          }

        default:
          fmt.Println("*** Unknown Spec Type:")
          ast.Print(fileset, spec.Type)
        }

      default:
        fmt.Println("*** Unknown Spec:")
        ast.Print(fileset, spec)
      }
    }
    fmt.Println("}\n")

  default:
    fmt.Println("*** Unknown Tok:")
    ast.Print(fileset, decl.Tok)
  }
}
```

```go
// Convert Go Struct Fields to Dart
func convertFields(fileset *token.FileSet, astFields []*ast.Field) {
  for _, field := range astFields {
    // ast.Print(fileset, field)
    dartField := convertField(fileset, field)
    if dartField.Name != "" {
      fmt.Printf("  %s %s;\t//  %s: %s\n", dartField.DartType, dartField.Name, dartField.CborName, dartField.GoType)
    }
  }
}
```

# Auto Generate CBOR Encoder

TODO

[`nmxact/nmp/image.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/master/nmxact/nmp/image.go)

```go
//  Convert From Go...
//  Go Struct
type ImageUploadReq struct {
  NmpBase  `codec:"-"`
  ImageNum uint8  `codec:"image"`
  Off      uint32 `codec:"off"`
  Len      uint32 `codec:"len,omitempty"`
  DataSha  []byte `codec:"sha,omitempty"`
  Upgrade  bool   `codec:"upgrade,omitempty"`
  Data     []byte `codec:"data"`
}
```

[`dart/nmp/image.dart`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/nmp/image.dart)

```dart
//  Converted To Dart...
//  Converted Dart Class
class ImageUploadReq 
  with NmpBase       //  Get and set SMP Message Header
  implements NmpReq  //  SMP Request Message
{
  ...
  /// Encode the SMP Request fields to CBOR
  void Encode(cbor.MapBuilder builder) {
    builder.writeString("image");
    builder.writeInt(ImageNum); // uint8
    builder.writeString("off");
    builder.writeInt(Off);      // uint32
    builder.writeString("len");
    builder.writeInt(Len);      // uint32
    builder.writeString("sha");
    builder.writeArray(DataSha);// []byte
    builder.writeString("upgrade");
    builder.writeBool(Upgrade); // bool
    builder.writeString("data");
    builder.writeArray(Data);   // []byte
  }
}
```

Here is the code that generates the CBOR Encoder for a Go Struct: [`dart/convert.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/convert.go)

```go
// Generate the Dart CBOR Encoder function for the Go Struct Fields
func generateCborEncoder(fileset *token.FileSet, astFields []*ast.Field) {
  fmt.Println("  /// Encode the SMP Request fields to CBOR")
  fmt.Println("  void Encode(cbor.MapBuilder builder) {")
  for _, field := range astFields {
    // ast.Print(fileset, field)
    dartField := convertField(fileset, field)
    if dartField.CborName != "-" { // Fields tagged `codec:"-"` will not be ended
      // Encode the string key
      fmt.Printf("    builder.writeString(\"%s\");\n", dartField.CborName)
      // Encode the value
      fmt.Printf("    builder.write%s(%s);\t// %s\n", dartField.CborType, dartField.Name, dartField.GoType)
    }
  }
  fmt.Println("  }")
}
```

# Auto Convert Go Function to Dart

TODO

[`nmxact/nmp/image.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/master/nmxact/nmp/image.go)

```go
//  Convert From Go...
//  Go Function
func NewImageUploadReq() *ImageUploadReq {
  r := &ImageUploadReq{}
  fillNmpReq(r, NMP_OP_WRITE, NMP_GROUP_IMAGE, NMP_ID_IMAGE_UPLOAD)
  return r
}
```

[`dart/nmp/image.dart`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/nmp/image.dart)

```dart
//  Converted To Dart...
//  Converted Dart Function
ImageUploadReq NewImageUploadReq() {
  var r = ImageUploadReq();
  fillNmpReq(r, NMP_OP_WRITE, NMP_GROUP_IMAGE, NMP_ID_IMAGE_UPLOAD);
  return r;
}
```

Here is the code that converts a Go function to Dart: [`dart/convert.go`](https://github.com/lupyuen/mynewt-newtmgr/blob/ast/dart/convert.go)

```go
// Convert Go Function to Dart
func convertFunction(fileset *token.FileSet, decl *ast.FuncDecl) {
  // ast.Print(fileset, decl)
  name := decl.Name                                               // Function Name: "NewImageUploadReq"
  returnType := fmt.Sprintf("%v", decl.Type.Results.List[0].Type) // Return Type: "&{40 ImageUploadReq}"

  // Convert the return type "&{40 ImageUploadReq}" to "ImageUploadReq"
  if strings.HasPrefix(returnType, "&{") && strings.HasSuffix(returnType, "}") {
    returnType = strings.Split(returnType, " ")[1]
    returnType = strings.Replace(returnType, "}", "", 1)
  }
  // TODO: Convert function parameters

  // Output function declaration
  fmt.Printf("%s %s() {\n", returnType, name)

  // Convert the statements in the body
  body := decl.Body.List
  for _, stmt := range body {
    // ast.Print(fileset, stmt)
    // Convert the statement to a string
    var buf bytes.Buffer
    if err := format.Node(&buf, fileset, stmt); err != nil {
      panic(err)
    }
    dartStmt := fmt.Sprintf("%s", buf.Bytes()) // "r := &ImageUploadReq{}"

    // Convert specific kinds of statements
    switch stmt.(type) {
    case *ast.AssignStmt:
      // For Assignment Statement "r := &ImageUploadReq{}", rewrite to "var r = ImageUploadReq()"
      dartStmt = strings.Replace(dartStmt, ":=", "=", 1)
      dartStmt = strings.Replace(dartStmt, "&", "", 1)
      dartStmt = strings.Replace(dartStmt, "{}", "()", 1)
      dartStmt = "var " + dartStmt
    }

    // Terminate every statement with semicolon
    fmt.Printf("  %s;\n", dartStmt)
  }
  // End of function
  fmt.Println("}\n")
}
```

# Test Auto Conversion

TODO

```
> Executing task: /usr/local/go/bin/go run dart/convert.go <

//  Go Code...

package main

type ImageUploadReq struct {
        NmpBase  `codec:"-"`
        ImageNum uint8  `codec:"image"`
        Off      uint32 `codec:"off"`
        Len      uint32 `codec:"len,omitempty"`
        DataSha  []byte `codec:"sha,omitempty"`
        Upgrade  bool   `codec:"upgrade,omitempty"`
        Data     []byte `codec:"data"`
}

func NewImageUploadReq() *ImageUploadReq {
        r := &ImageUploadReq{}
        fillNmpReq(r, NMP_OP_WRITE, NMP_GROUP_IMAGE, NMP_ID_IMAGE_UPLOAD)
        return r
}

//  Converted To Dart...

class ImageUploadReq 
  with NmpBase       //  Get and set SMP Message Header
  implements NmpReq  //  SMP Request Message
{
  int ImageNum; //  image: uint8
  int Off;      //  off: uint32
  int Len;      //  len: uint32
  typed.Uint8Buffer DataSha;    //  sha: []byte
  bool Upgrade; //  upgrade: bool
  typed.Uint8Buffer Data;       //  data: []byte

  NmpMsg Msg() { return MsgFromReq(this); }

  /// Encode the SMP Request fields to CBOR
  void Encode(cbor.MapBuilder builder) {
    builder.writeString("image");
    builder.writeInt(ImageNum); // uint8
    builder.writeString("off");
    builder.writeInt(Off);      // uint32
    builder.writeString("len");
    builder.writeInt(Len);      // uint32
    builder.writeString("sha");
    builder.writeArray(DataSha);        // []byte
    builder.writeString("upgrade");
    builder.writeBool(Upgrade); // bool
    builder.writeString("data");
    builder.writeArray(Data);   // []byte
  }
}

ImageUploadReq NewImageUploadReq() {
  var r = ImageUploadReq();
  fillNmpReq(r, NMP_OP_WRITE, NMP_GROUP_IMAGE, NMP_ID_IMAGE_UPLOAD);
  return r;
}
```

# What's Next

TODO

The code in this article is part of the upcoming open source [__PineTime Companion App__](https://github.com/lupyuen/pinetime-companion) for Android and iOS. So that we can update the firmware on our PineTime Smart Watches wirelessly, sync the date and time, show notifications from our phone, chart our heart rate, ... Maybe even control our smart home gadgets! 

We'll be doing lots more coding...

1. __Handle Other PineTime Commands:__ Update firmware, sync date and time, show mobile notifications, control smart home gadgets (via IFTTT and MQTT), ...

    We shall do this by taking the [Newt Manager](https://github.com/apache/mynewt-newtmgr) code in Go and converting it to Flutter and Dart, as explained here...

    _["Convert Go to Flutter and Dart for PineTime Companion App"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/companion)_

1. __Companion App for Linux Phones (like PinePhone):__ We shall take the [Newt Manager](https://github.com/apache/mynewt-newtmgr) code in Go and wrap it into a GTK3 app, using the [`gotk3` library](https://github.com/gotk3/gotk3)...

    _["Your First GTK App with Go and VSCodium"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/gotk3)_

1. __PineTime Firmware Support:__ Today our PineTime Companion App talks to Mynewt and Zephyr operating systems on PineTime. We hope to implement the same Bluetooth LE protocol (Simple Management Protocol) on other operating systems, so that they may also enjoy wireless firmware updates...

    _["Firmware Update over Bluetooth Low Energy on PineTime Smart Watch"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/dfu)_

_Why are we maintaining two code bases: Flutter (for Android and iOS) and Go (for Linux phones)?_

Because Flutter is probably the best way to build mobile apps... But it's not officially supported for Linux phones.  The `flutter_blue` plugin doesn't support Linux either.  

So we need to stick with Go for Linux phones.

We're now exploring [__`go-flutter`__](https://github.com/go-flutter-desktop/go-flutter) for porting the Flutter App to Linux. And recode `flutter_blue` via FFI to a Linux Bluetooth LE library (in Go or C).

_(Maybe someday when Flutter is officially supported on Linux phones, we can scrap the Go version!)_

If you're keen to help out with the PineTime Companion App (or anything else in PineTime), __come chat with the PineTime FOSS Community (and me) in the PineTime Chatroom!__

[PineTime Chatroom on Matrix / Discord / Telegram / IRC](https://wiki.pine64.org/index.php/PineTime#Community)

# Further Reading

_["Flutter State Management with Bloc for PineTime Companion App"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/bloc)_

_["Convert Go to Flutter and Dart for PineTime Companion App"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/companion)_

_["Your First Bluetooth Low Energy App with Flutter"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/flutter)_

_["Your First GTK App with Go and VSCodium"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/gotk3)_

_["MCUBoot Bootloader for PineTime Smart Watch (nRF52)"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/mcuboot)_

_["Firmware Update over Bluetooth Low Energy on PineTime Smart Watch"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/dfu)_

_["Wireless Firmware Update In Action on PineTime Smart Watch (nRF52)"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/dfutest)_

[Check out the other PineTime articles](https://github.com/lupyuen/pinetime-rust-mynewt/blob/master/README.md)

[RSS Feed](https://lupyuen.github.io/rss.xml)