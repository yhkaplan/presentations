slidenumbers: true
slide-transition: true
theme: Work
autoscale: true

# Using the latest UICollectionView APIs

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
* Prefetching

---

# Layout

---

## Definition of UICollectionView and what it’s for

- Standard Grid
- Comparison w/ UITableView
- Manages multiple scrolling views
    - **completely configurable** layout
    - High performance, view recycling

![80%right](assets/basic_grid.gif)

---

## Basic Grid

- UICollectionViewFlowLayout
- Define in code UICollectionViewFlowLayout or delegate, or interface builder

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
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: BasicCell.reuseID, for: indexPath)

        let systemName = data[indexPath.item]
        if let image = UIImage(systemName: systemName) {
            (cell as? BasicCell)?.configure(with: image)
        }
        return cell
    }
}
```

---

## Completely configurable🤔

- Not just basic grids!
- Lists, complex grids, 3D stacks, carousels, or anything

---

## Examples

---

## Spinner

- [jVirus/uicollectionview-layouts-kit](https://github.com/jVirus/uicollectionview-layouts-kit)

![80%right](assets/spinner.gif)

---

## Safari

- [jVirus/uicollectionview-layouts-kit](https://github.com/jVirus/uicollectionview-layouts-kit)

![80%right](assets/safari.gif)

---

## Carousel

- [zepojo/UPCarouselFlowLayout](https://github.com/zepojo/UPCarouselFlowLayout)

![80%right](assets/carousel.gif)

---

## BounceyLayout

- [GitHub - roberthein/BouncyLayout: Make. It. Bounce.](https://github.com/roberthein/BouncyLayout)

![80%right](assets/bounce.gif)

---

## List

- iOS 14+
- Part of CompositionalLayout
- All main styles available

![inline, fit](assets/list_plain.png) ![inline, fit](assets/list_grouped.png) ![inline, fit](assets/list_inset_grouped.png)

---

```swift
final class ListVC: UIViewController {
    enum Section: Hashable { case list }

    private let data = ["pencil", "trash", "paperplane", "calendar", "lightbulb"]

    private lazy var collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
    private lazy var layout: UICollectionViewCompositionalLayout = {
        let config = UICollectionLayoutListConfiguration(appearance: .plain)
        return UICollectionViewCompositionalLayout.list(using: config)
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

    override func viewDidLoad() {...}
}
```

---

## CompositionalLayout

- Complex, grouped sections
- Convenient for future proofing

![fit|right](assets/compositional_layout.jpeg)

---

## Code

```swift
final class BasicCompositionalLayoutGridVC: UIViewController {
    enum Section: Hashable { case grid }

    private let data = ["pencil", "trash", "paperplane", "calendar", "lightbulb"]

    private lazy var collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
    private lazy var layout = UICollectionViewCompositionalLayout { sectionIndex, environment in
        let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.5), heightDimension: .fractionalHeight(1.0))
        let item = NSCollectionLayoutItem(layoutSize: itemSize)
        item.contentInsets = NSDirectionalEdgeInsets(top: 4, leading: 4, bottom: 4, trailing: 4)

        let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .fractionalWidth(0.5))
        let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])

        return NSCollectionLayoutSection(group: group)
    }
    private lazy var dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: collectionView) {...}

    override func viewDidLoad() {...}
}
```

---

## Custom Layout

---

// TODO: code sample

---

## UIKitDynamics

// TODO: code sample

---

# Data and Cell Configuration

---

## DiffableDataSources

- Fits most cases
- iOS 14 and 15 added // TODO: x
- Animation behavior difficult to customize
- [SE-0240: Ordered Collection Diffing](https://github.com/apple/swift-evolution/blob/main/proposals/0240-ordered-collection-diffing.md) is your friend
    - Find inserted, deleted, and updated items/sections, then simply use [`performBatchUpdates(_:completion:)`](https://developer.apple.com/documentation/uikit/uicollectionview/1618045-performbatchupdates)

---

## Code

```swift
final class BasicCompositionalLayoutGridVC: UIViewController {
    enum Section: Hashable { case grid }

    private let data = ["pencil", "trash", "paperplane", "calendar", "lightbulb"]

    private lazy var collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
    private lazy var layout = UICollectionViewCompositionalLayout {...}
    private lazy var dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: collectionView) { collectionView, indexPath, item in
        let registration = UICollectionView.CellRegistration<UICollectionViewCell, String> { cell, indexPath, item in
            let image = UIImage(systemName: item)
            cell.contentConfiguration = ImageContentView.Config(image: image)
        }

        return collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: item)
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        view.addSubview(collectionView)

        var snapshot = NSDiffableDataSourceSnapshot<Section, String>()
        snapshot.appendSections([.grid])
        snapshot.appendItems(data, toSection: .grid)
        dataSource.apply(snapshot, animatingDifferences: false)
    }
}
```

---

## Cell configuration and updating

- Useful for UITableView-style cells
- Custom cells maybe more work than preferable

---

## List example

```swift
final class ListVC: UIViewController {
    enum Section: Hashable { case list }

    private let data = ["pencil", "trash", "paperplane", "calendar", "lightbulb"]

    private lazy var collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
    private lazy var layout: UICollectionViewCompositionalLayout = {
        let config = UICollectionLayoutListConfiguration(appearance: .plain)
        return UICollectionViewCompositionalLayout.list(using: config)
    }()
    private lazy var dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: collectionView) { collectionView, indexPath, item in
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

    override func viewDidLoad() {...}
}
```
---

# Custom example

---

## UIContentView

```swift
final class ImageContentView: UIView, UIContentView {
    private var _configuration: Config
    private let imageView = UIImageView()

    var configuration: UIContentConfiguration {
        get { _configuration }
        set {
            guard let config = newValue as? Config else { return }
            _configuration = config
            imageView.image = _configuration.image
        }
    }

    init(config: Config) {
        _configuration = config
        super.init(frame: .zero)

        backgroundColor = .lightGray

        layoutImageView()
        imageView.image = _configuration.image
    }

    private func layoutImageView() {...}

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    struct Config: UIContentConfiguration {
        let image: UIImage?

        func makeContentView() -> UIView & UIContentView { ImageContentView(config: self) }
        func updated(for state: UIConfigurationState) -> ImageContentView.Config { self }
    }
}
```

## Registration

```swift
final class BasicCompositionalLayoutGridVC: UIViewController {
    enum Section: Hashable { case grid }

    private let data = ["pencil", "trash", "paperplane", "calendar", "lightbulb"]

    private lazy var collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
    private lazy var layout = UICollectionViewCompositionalLayout {...}
    private lazy var dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: collectionView) { collectionView, indexPath, item in
        let registration = UICollectionView.CellRegistration<UICollectionViewCell, String> { cell, indexPath, item in
            let image = UIImage(systemName: item)
            cell.contentConfiguration = ImageContentView.Config(image: image)
        }

        return collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: item)
    }

    override func viewDidLoad() {...}
}
```

^ More thorough example with different content and background configurations on Apple's developer website

---

## Updating

// TODO:
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
- Consider using SwiftUI.List too

---

## Conclusion

- Find the right fit
- Older ways and SwiftUI work too
- You mostly don't need UITableView anymore
- UICollectionView is _really_ flexible and performant
- SwiftUI's LazyGrid views are still somewhat lacking in comparison

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

* 2016 or earlier / iOS 10
	* DataSourcePrefetching
* 2017 or earlier / iOS 11
	* drag and drop
* 2019 / iOS 13
	* DiffableDataSource
	* CompositionalLayout
* 2020 / iOS 14
	* cell configuration
	* UICollectionView list style
	* DiffableDataSource & CompositionalLayout new features
* 2021 / iOS 14.5
	* UIListSeparatorConfiguration
* 2021 / iOS 15
	* cell configuration new features
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