# Swift代码规范

## 一、命名
### 1.类型
* 类型名称（如 struct, enum, class, typedef, associatedtype, protocol 等）使用**大驼峰命名法**命名。
* 变量和常量则以**小驼峰命名法**命名。

```Swift
enum Planet {
    case mercury, venus, earth, mars, jupiter, saturn, uranus, neptune
}

struct Person {
    var mobileNo: String
}

```

### 2.协议
根据苹果接口设计指导准则，协议名称用来描述一些东西是什么的时候是名词，例如：Collection, WidgetFactory。若协议名称用来描述能力应该以 -ing, -able, 或 -ible 结尾，例如：Equatable, Resizing。

### 3.类前缀
* Swift中类别(类，结构体)在编译时会把模块设置为默认的命名空间，所以不用为了区分类别而添加前缀，比如RW。
* 如果担心来自不同模块的两个名称发生冲突，可以在使用时添加模块名称来区分。注意不要滥用模块名称，仅在有可能发生冲突或疑惑的场景下使用。如：

```
import SomeModule 
let myClass = SomeModule.UsefulClass()
```

### 4.参数命名
#### 委托
在定义委托方法时，第一个未命名参数应是委托数据源。  
正确代码：

```
func namePickerView(_ namePickerView: NamePickerView, didSelectName name: String)

func namePickerViewShouldReload(_ namePickerView: NamePickerView) -> Bool
```

错误代码：

```
func didSelectName(namePicker: NamePickerViewController, name: String)

func namePickerShouldReload() -> Bool
```

#### 泛型
泛型类参数应具有描述性，遵守“大骆驼命名法”。如果一个参数名没有具体的含义，可以使用传统单大写字符，如T,  U, 或V等。
正确代码：

```
struct Stack<Element>{ ... }
func writeTo <Target: OutputStream>(to Target: inout Target)
func swap(_ a: inout T, _ b: inout T) 
```

错误代码：

```
struct Stack<T>{ ... }
func write<target: OutputStream> (to target: inout target)
func swap<Thing>(_ a: inout Thing, _ b: inout Thing)
```


## 二、代码逻辑
### 1.类型推断
尽量使用类型推断以减少多余的冗余类型信息，例如：  
正确的写法：

```
let currentLocation = Location()
```

错误的写法：

```
let currentLocation: Location = Location()
```

### 2.self推断
* 让编译器在所有允许的地方推断 self 。
* 在 init 中设置参数时显性地使用 self 。
* 在 non-escaping closures 中显性地使用 self 。
* 在避免命名冲突时使用 self 。  
例：

```
class BoardLocation  {
    let row: Int, column: Int
    init(row: Int, column: Int)  {
        self.row = row
        self.column = column
        let closure = {
            print(self.row)    
         }  
    }
}
```

### 3.常量
* 类常量应该在类型里声明为`static`。使用`static`修饰常量可以允许他们在被引用的时候不需要实例化类型。
* 除了单例以外，应尽量避免生成全局常量。
* 定义常量使用 **let** 关键字，定义变量使用 **var** 关键字， 如果变量的值未来不会发生变化要使用常量。

```
struct PhysicsModel {
    static var speedOfLightInAVacuum = 299_792_458
}
 
class Spaceship {
    static let topSpeed = PhysicsModel.speedOfLightInAVacuum
    var speed: Double
     
    func fullSpeedAhead() {
        speed = Spaceship.topSpeed
    }
}
```

### 4.计算型类型属性
* 当只需要继承 getter 方法时，返回简单的 Computed 属性即可。
  
正确代码：

```
class Example {
    var age: UInt32 {
        return arc4random()
    }
}
```

错误代码：

```
class Example {
    var age: UInt32 {
        get {
            return arc4random()
        }
    }
}
```

* 如果你在属性中添加了 set 或者 didSet ，那么你应该显示地提供 get 方法。

```
class Person {
    var age: Int {
        get {
            return Int(arc4random())
        }
        set {
            print("That's not your age.")
        }
    }
}
```

### 5.单例
Swift中单例很简单，Swift 的 runtime 会保证单例的创建并且采用线程安全的方式访问：

```
class ControversyManager {
    static let sharedInstance = ControversyManager()
}

```
单例通常只需要访问"sharedInstance"的静态属性，除非你有不得已的原因去重命名它。注意，不要用静态函数或者全局函数去访问你的单例。

### 6.错误处理
可以使用`do/try/catch`机制，避免使用`try!`和`try?`

### 7.可选值
* 声明一个函数的某个参数可以为 nil 时，用`？`
* 当你确定某个变量在使用时已经确定不是 nil 时，在后面加`!`

### 8.扩展声明周期
用 [weak self] 和 guard let strongSelf = self else { return } 模式扩展生命周期。用 [weak self] 比 [unowned self] 更好。  
正确代码：

```
resource.request().onComplete { [weak self] response in
  guard let strongSelf = self else { return }
  let model = strongSelf.updateModel(response)
  strongSelf.updateUI(model)
}
```

错误代码：

```
// might crash if self is released before response returns
resource.request().onComplete { [unowned self] response in
  let model = self.updateModel(response)
  self.updateUI(model)
}
```

```
// deallocate could happen between updating the model and updating UI
resource.request().onComplete { [weak self] response in
  let model = self?.updateModel(response)
  self?.updateUI(model)
}
```

## 三、代码结构
### 1.注释
* 使用Xcode8自带的注释功能，快捷键`Option+Command+/`
* 使用`// MARK:`分隔代码（类似于OC中的`#pragma mark`） 

### 2.缩进
遵守Xcode内置的缩进格式
 
### 3.类型 
优先使用Swift原生类型，可以根据需要使用Objective-C提供的方法，因为Swift提供了到Objective-C的桥接 。  
正确代码：

```
let width = 120.0  // Double
let widthString = (width as NSNumber).stringValue // String
```

错误代码：

```
let width: NSNumber = 120.0 // NSNumber
let widthString: NSString=width.stringValue  // NSString
```


### 4.协议一致性
当一个对象要实现协议一致性时，推荐使用 **extension** 隔离协议中的方法集，这样让相关方法和协议集中显示在一起，也简化了类支持一个协议和实现相关方法的流程。  
正确代码：

```
class MyViewcontroller: UIViewController {
      // 方法
} 

extension MyViewcontroller: UITableViewDataSource {
      // UITableViewDataSource 方法
}

extension MyViewcontroller: UIScrollViewDelegate {
      // UIScrollViewDelegate 方法
} 
```  

错误代码：

```
class MyViewcontroller: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
       // 所有的方法
}
```

### 5.良好的类定义
* 属性，变量，常量和参数等在声明定义时，其中: 符号后有空格，而: 符号前没有空格，比如x: Int, 和Circle: Shape
* 定义多个变量/数据结构时，出于相同的目的和上下文，可以定义在同一行。
* 缩进 getter，setter 的定义和属性观察器的定义。
* 不需要添加`internal`这样的默认的修饰符。也不需要在重写一个方法时添加访问修饰符。
* 给那些不打算被继承的类使用`final`修饰符

```
final class Circle: Shape {
  var x: Int, y: Int
  var radius: Double
  var diameter: Double{
      get {
        returnradius * 2
     } 
     set {      
        radius=newValue/2
    }  
  }
  init(x: Int, y: Int, radius: Double) {
      self.x = x
      self.y = y
      self.radius = radius  
  }

  convenience init(x: Int, y: Int, diameter: Double) {
     self.init(x: x, y: y, radius: diameter/2)  
  }

  func describe() -> String{
    return"I am a circle at\(centerString())with an        area of\(computeArea())"
  }
  override func computeArea() -> Double{
    return M_PI * radius * radius  
  }

  private func centerString()->String{
    return "(\(x),\(y))"
  }
}
```

### 6.控制流程
* 循环使用for-in表达式，而不使用 while 表达式。  
正确代码：

```
for _ in 0..<3 {
  print("Hello three times")
}

for(index, person) in attendeeList.enumerate() {
    print("\(person)is at position #\(index)")
}

for index in 0.stride(from: 0, to: items.count, by: 2) {
  print(index)
}
for index in (0...3).reverse() {
    print(index)
}
```

错误代码：

```
var i=0 
while i<3 {
  print("Hello three times") 
  i+=1
}

var i=0
while i<3 {
  let person = attendeeList[i]
  print("\(person)is at position #\(i)")  
  i+=1
}
```

* Switch 模块中不用显式使用break。

### 7.黄金路径
当编码遇到条件判断时，左边的距离是黄金路径或幸福路径，因为路径越短，速度越快。不要嵌套if循环，多个返回语句是可以的。guard 就为此而生的。  
正确代码：

```
func computeFFT(context: Context?, inputData: InputData?)  throws -> Frequencies {
    guard let context = context else {throwFFTError.NoContext }

   guard let inputData = inputData else { throwFFTError.NoInputData }

  //计算frequencies
  return frequencies
}
```

错误代码：

```
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {
if let context = context {
    if let inputData = inputData {
           // 计算frequencies
          return frequencies    
    } else {
         throwFFTError.NoInputData    
     }  
   } else {
      throwFFTError.NoContext  
   }
}
```

当有多个条件需要用 guard 或 if let 解包，可用复合语句避免嵌套。

```
guard let number1=number1, number2=number2, number3=number3 else{
    fatalError("impossible") 
}
// 处理number
```

### 8.语法糖
```
//对空的数据和字典，使用类型注解
var names:  [String] = []
var lookup:  [String: Int] = [:]

var deviceModels: [String]
var employees: [Int:String]
var faxNumber: Int?

```

### 9.函数
自由函数不依附于任何类或类型，应该节制地使用。  
正确代码：

```
let sorted = items.mergeSort() // 易发现性
rocket.launch()  // 可读性
```

错误代码：

```
let sorted = mergeSort(items)// 不易被发现
launch(&rocket)
```

天然的自由函数：

```
let value = max(x,y,z)  // another free function that feels natural
```


参考链接：  
[最详尽的 Swift 代码规范指南](http://www.cocoachina.com/swift/20160725/17176.html)  
[Swift 3.0 编码规范](http://www.jianshu.com/p/583025f702a8)  
[17条 Swift 最佳实践规范](http://www.cocoachina.com/swift/20151010/13664.html)  
[一份比较通用的iOS代码规范，包括Objective-C和Swift](https://github.com/ValiantCat/iOS-Code-Style)  
[Swift 编码规范（中文）](http://www.jianshu.com/p/288ea00dcfcf)
