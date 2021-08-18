## Using the latest UICollectionView APIs

---

## TOC

* Intro
* Topics skipped
* Layout
* Data and Cell Configuration
* Future Directions
* References and Miscellaneous 

---

## Intro

- Joshua Kaplan (`yhkaplan`)
- Senior iOS Engineer @ GMO Pepabo working on *minne*
- Into tooling, frameworks, and architecture
- Stay-at-home shiba-inu parent

---

## Topics skipped

* Drag and drop
* Cell editing
* Headers and footers

---

# Layout

---

## Definition of UICollectionView and what it’s for

- Standard Grid
- Comparison w/ UITableView
- Manages multiple scrolling views, with **completely configurable layout**

// TODO: screenshot

---

## Basic Grid

- UICollectionViewFlowLayout

---

```swift
final class CollectionViewBasicsVC: UIViewController {
    private lazy var data = (0...100).compactMap { _ in
        ["pencil", "trash", "paperplane", "calendar", "lightbulb"].randomElement()
    }
    private lazy var layout: UICollectionViewFlowLayout = {
        let l = UICollectionViewFlowLayout()
        let halfWidth = view.bounds.width / 2
        let halfWidthMinusMargins = halfWidth - 14
        let height = halfWidthMinusMargins
        l.itemSize = CGSize(width: halfWidthMinusMargins, height: height)
        return l
    }()
    private lazy var collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)


    override func viewDidLoad() {
        super.viewDidLoad()

        view.addSubview(collectionView)

        collectionView.contentInset = UIEdgeInsets(top: 8, left: 8, bottom: 8, right: 8)
        collectionView.backgroundColor = .white
        collectionView.register(BasicCell.self, forCellWithReuseIdentifier: BasicCell.reuseID)
        collectionView.dataSource = self
    }
}

extension CollectionViewBasicsVC: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        data.count
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: BasicCell.reuseID,
            for: indexPath
        )

        let systemName = data[indexPath.item]
        if let image = UIImage(systemName: systemName) {
            (cell as? BasicCell)?.configure(with: image)
        }
        return cell
    }
}
```

---

## Completely configurable ��

- Not just basic grids!
- Lists, complex grids, 3D stacks, carousels, or anything

---

## Examples

---

## List

- iOS 14+
- Part of CompositionalLayout

// TODO: screenshot

---

```swift
final class ListVC: UIViewController {
    enum Section: Hashable { case list }

    private let data = ["pencil", "trash", "paperplane", "calendar", "lightbulb"]

    private lazy var collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
    private lazy var layout: UICollectionViewCompositionalLayout = {
        let config = UICollectionLayoutListConfiguration(appearance: .plain)
        let l = UICollectionViewCompositionalLayout.list(using: config)
        return l
    }()
    private lazy var dataSource = UICollectionViewDiffableDataSource<Section, String>(
        collectionView: collectionView
    ) { collectionView, indexPath, item in
        let registration = UICollectionView.CellRegistration<UICollectionViewListCell, String> { cell, indexPath, item in
            var content = cell.defaultContentConfiguration()
            content.image = UIImage(systemName: item)
            content.text = item
            cell.contentConfiguration = content
        }

        let cell = collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: item)
        cell.accessories = [.disclosureIndicator()]

        return cell
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        view.addSubview(collectionView)

        var snapshot = NSDiffableDataSourceSnapshot<Section, String>()
        snapshot.appendSections([.list])
        snapshot.appendItems(data, toSection: .list)
        dataSource.apply(snapshot, animatingDifferences: false)
    }
}
```

---

## CompositionalLayout

// TODO: screenshot


---

## Code

// TODO: code

---

## Custom Layout

---

// TODO: GIFs

---

// TODO: code sample

---

## UIKitDynamics

// TODO: code sample

---

# Data and Cell Configuration

---

## Prefetching

---

## DiffableDataSources

- Animation behavior difficult to customize
- [SE-0240: Ordered Collection Diffing](https://github.com/apple/swift-evolution/blob/main/proposals/0240-ordered-collection-diffing.md) is your friend

---

## Cell configuration and updating

* https://developer.apple.com/documentation/uikit/uicollectionviewcell/3751733-configurationupdatehandler
* https://jessesquires.github.io/wwdc-notes/2021/10252_blazing_fast_collection_views.html

---

# Future Directions

---

## SwiftUI

- List and Grid
- Performance
- Customizability

---

## When to use UITableView

- complex list-style customization
- self-sizing cells w/ dynamic height are generally easier
- Consider using SwiftUI.List

---

## Conclusion

---

## References and Miscellaneous 
- Transitions and dynamically changing layout

### OSS Examples

* Various layouts
    * [jVirus/uicollectionview-layouts-kit](https://github.com/jVirus/uicollectionview-layouts-kit)
    * [amirdew/CollectionViewPagingLayout](https://github.com/amirdew/CollectionViewPagingLayout)
* Abstracting many layouts [WenchaoD/FSPagerView](https://github.com/WenchaoD/FSPagerView)
* Neat slanted layout [yacir/CollectionViewSlantedLayout](https://github.com/yacir/CollectionViewSlantedLayout)
* Abstraction based on delegates [airbnb/MagazineLayout](https://github.com/airbnb/MagazineLayout)

### UICollectionViewFlowLayout
* Easier than doing a custom layout from scratch
* Demo of making a custom layout like Pinterest
* 簡単なレイアウトなら、実装コストがより低いので、完全にカスタムなレイアウトを使うより好ましい

### Examples
* Pinterest-like layout [ChernyshenkoTaras/SquareFlowLayout](https://github.com/ChernyshenkoTaras/SquareFlowLayout)
* Carousel layout: [zepojo/UPCarouselFlowLayout](https://github.com/zepojo/UPCarouselFlowLayout)
*  UIKitDynamics [GitHub - roberthein/BouncyLayout: Make. It. Bounce.](https://github.com/roberthein/BouncyLayout)

### CompositionalLayout

### Comparison w/ SwiftUI.Grid
* Performance 
* Custom scrolling and animation behavior 
* Unusual layouts

* UIScrollViewDelegate

### Timeline

* 2016年以前 / iOS 10
	* DataSourcePrefetching
* 2017年以前 / iOS 11
	* drag and drop
* 2019 / iOS 13
	* DiffableDataSource
	* CompositionalLayout
* 2020 / iOS 14
	* cell configuration
	* UICollectionViewのリストスタイル
	* DiffableDataSourceとCompositionalLayoutの進化
* 2021 / iOS 14.5
	* UIListSeparatorConfiguration
* 2021 / iOS 15
	* cell configurationの進化
	*  DiffableDataSourceとCompositionalLayoutのさらなる進化

## Reference

* https://developer.apple.com/videos/play/wwdc2012/219/
* UIKit Dynamics
	* https://developer.apple.com/videos/play/wwdc2013/206/
	* https://developer.apple.com/videos/play/wwdc2013/221/
* https://developer.apple.com/documentation/uikit/uicollectionviewlayout
* [About iOS Collection Views](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40012334-CH1-SW1)

## Miscellaneous
* Change layout dynamically
> To create an interactive transition—one that is driven by a gesture recognizer or touch events—use the  [startInteractiveTransition(to:completion:)](https://developer.apple.com/documentation/uikit/uicollectionview/1618098-startinteractivetransition)  method to change the layout object. That method installs an intermediate layout object, which works with your gesture recognizer or event-handling code to track the transition progress. When your event-handling code determines that the transition is finished, it calls the  [finishInteractiveTransition()](https://developer.apple.com/documentation/uikit/uicollectionview/1618080-finishinteractivetransition)  or  [cancelInteractiveTransition()](https://developer.apple.com/documentation/uikit/uicollectionview/1618075-cancelinteractivetransition)  method to remove the intermediate layout object and install the intended target layout object.  
* https://developer.apple.com/documentation/uikit/uicollectionview/1618017-setcollectionviewlayout
* https://developer.apple.com/videos/play/wwdc2021/10252/