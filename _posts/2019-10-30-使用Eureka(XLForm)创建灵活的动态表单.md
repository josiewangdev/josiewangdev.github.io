---
layout: post
title: 使用Eureka(XLForm)创建灵活的动态表单
tags: Project
categories: Tech
cover: "upload/blog_masonry_03.jpg"
---

做iOS前端开发的童鞋应该都知道，要实现一个列表样式，我们需要实现TableView的数据源和代理方法（UITableViewDataSource, UITableViewDelegate）, 通常情况下，对于动态从后台获取的数据表单来说，这样的实现无可厚非。但是，我们经常也会使用到一些复杂的静态表单，这个时候如果再去一一定义每个TableViewCell的样式，组装数据源结构，就未免有些太过繁琐与冗杂。不过，通过下面这个第三方库就能够使用灵活的控件与整洁的代码结构去创建一个多功能的静态表单。

<br>
### Eureka (Swift版，OC版本的第三方库叫做XLForm)
<br>

在App中经常会有让用户填写表单或者输入信息的表单样式。例如，如果要实现这样一个表单样式，用Eureka能以规整简洁的代码结构实现。

<img src="/images/eureka/eureka.gif" />

{% highlight swift %}

fileprivate func setupUI() {

    //创建一个表单
    form
    //添加一个单元组
    +++ Section()
    //添加行（该行为我创建的自定义选择相册图片样式）
    <<< MerchandiseAddMainImageRow(addImageRowTag) {[weak self] row in

        //数据是否有效，其中ImageValues是我对该组件定义的值类型
        let ruleRequired = RuleClosure<ImageValues> { rowValue in
            return (rowValue == nil || rowValue!.imageUrls.isEmpty) ? ValidationError(msg: "请至少选择一张图片") : nil
        }
        row.add(rule: ruleRequired)
        
        let imageUrls: String = []
        row.value = ImageValues(imageUrls: imageUrls)

        //协议代理实现按钮点击事件
        row.delegate = self

    }

    //另添加一单元组
    +++ Section()

    //添加自定义按钮行样式
    <<< GBFormButtonRow(productBrandRowTag) { row in
    
        //行标题
        row.mainTitle = "商品品牌"
        //自定义改行数据是否必填标志
        row.isRequired = true
        //行数据符合规则判断
        let ruleRequired = RuleClosure<String> { rowValue in
            return (rowValue == nil || rowValue!.isEmpty) ? ValidationError(msg: "请选择商品品牌") : nil
        }
        row.add(rule: ruleRequired)
        row.displayValue = "Display Value"
        row.value = "Some Value"
        
    }.onCellSelection({ (cell, row) in
        //在此实现点击选择该行之后需要执行的代码
    })

    //添加自定义文字行样式
    <<< GBFormTextRow(productNameRowTag) { row in

        row.mainTitle = "商品名称"
        //自定义的文字字数提示
        row.hintUnit = "/60"
        row.isRequired = true
        let ruleRequired = RuleClosure<String> { rowValue in
            return (rowValue == nil || rowValue!.isEmpty) ? ValidationError(msg: "请填写商品名称") : nil
        }
        row.add(rule: ruleRequired)
        row.value = "productFullName"
        
    }
}

{% endhighlight %}

自定义选择相册图片样式ImageRow代码示例

{% highlight swift %}

//定义该行的值类型
struct ImageValues: Equatable {

    var imageUrls: [String] = []

    //自定义 row value 必须实现该方法以判断值是否相同
    static func ==(lhs: ImageValues, rhs: ImageValues) -> Bool {
        return lhs.imageUrls == rhs.imageUrls
    }
}

class MerchandiseAddMainImageCell: Cell<ImageValues>, CellType {

    @IBOutlet weak var imageSlideView: UIView!
    @IBOutlet weak var imageSlideShow: ImageSlideshow!

    var addMainImageRow: MerchandiseAddMainImageRow {
        return row as! MerchandiseAddMainImageRow
    }

    required init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }

    override func setup() {
        super.setup()
        //自定义行高度
        height = {return 250}
        imageSlideShow.contentScaleMode = .scaleAspectFill
    }

    override func update() {
        super.update()
    }

    @IBAction func addMainImageButtonClicked() {
        addMainImageRow.delegate?.didClickedAddImageButton()
    }

    @IBAction func editMainImageButtonClicked() {
        addMainImageRow.delegate?.didClickedEditImageButton()
    }

}

//自定义协议方法
protocol MerchandiseAddMainImageRowDelegate {
    func didClickedAddImageButton()
    func didClickedEditImageButton()
}

final class MerchandiseAddMainImageRow: Row<MerchandiseAddMainImageCell>, RowType {

    var delegate: MerchandiseAddMainImageRowDelegate?

    override var value: ImageValues? {
        didSet {
            if let imageValue = value {
                if imageValue.imageUrls.count > 0 {
                    cell.imageSlideView.isHidden = false
                    let imageInput = imageValue.imageUrls.map({ (imgUrlStr) -> SDWebImageSource in
                        return SDWebImageSource(url: imgUrlStr.toURL(), placeholder: UIImage.defaultProductImage)
                    })
                    cell.imageSlideShow.setImageInputs(imageInput)
                }else {
                    cell.imageSlideView.isHidden = true
                }
            }
        }
    }

    required init(tag: String?) {
        super.init(tag: tag)
        //用自定义Nib文件设置行样式
        cellProvider = CellProvider<MerchandiseAddMainImageCell>(nibName: kMerchandiseAddMainImageCell)
    }
}

{% endhighlight %}

自定义文字行样式TextRow

{% highlight swift %}

//文字类型
enum GBFormTextRowType {
    case plainText,
         number
}

class GBFormTextCell: Cell<String>, CellType{

    @IBOutlet weak var asteriskLabel: UILabel!
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var textField: UITextField!
    @IBOutlet weak var hintLabel: UILabel!
    @IBOutlet weak var hintUnitLabel: UILabel!

    required init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }

    override func setup() {

        super.setup()
        height = {44}
        textField.addTarget(self, action: #selector(self.textDidChanged(inTextField:)), for: .editingChanged)

        let textRow = row as! GBFormTextRow
        titleLabel.text = textRow.mainTitle
        hintUnitLabel.text = textRow.hintUnit
        
        //是否必填
        if textRow.isRequired {
            asteriskLabel.isHidden = false
        }else {
            asteriskLabel.isHidden = true
        }

        switch textRow.rowType {
            case .plainText:
                textField.textAlignment = .left
                textField.keyboardType = .default
                hintLabel.isHidden = false
            case .number:
                textField.textAlignment = .right
                textField.keyboardType = .decimalPad
                hintLabel.isHidden = true
        }

    }

    override func update() {
        super.update()
        textField.text = row.value
        textField.isEnabled = !(row.isDisabled)
        hintUnitLabel.text = (row as! GBFormTextRow).hintUnit
    }

    //MARK: - Event Response

    func textDidChanged(inTextField textField: UITextField) {
        hintLabel.text = "\(textField.text?.characters.count ?? 0)"
        row.value = textField.text
    }

}

final class GBFormTextRow: Row<GBFormTextCell>, RowType {

    var mainTitle: String?
    var hintUnit: String?
    var isRequired = false
    var rowType: GBFormTextRowType = .plainText

    required init(tag: String?) {
        super.init(tag: tag)
        cellProvider = CellProvider<GBFormTextCell>(nibName: "GBFormTextCell")
    }

}

{% endhighlight %}

[[参考Github地址] (OC) XLForm](https://github.com/xmartlabs/XLForm)

[[参考Github地址] (Swift) Eureka](https://github.com/xmartlabs/Eureka)


