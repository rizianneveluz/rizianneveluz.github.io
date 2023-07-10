---
layout: post
title: "Custom section header views in UICollectionView using UICollectionViewDiffableDataSource"
date: 2023-07-10 17:35:59 +0900
categories: Swift UICollectionViewDiffableDataSource UICollectionView
keywords: Swift, UICollectionViewDiffableDataSource, UICollectionView, custom header
---

There are 5 key steps to using a custom section header view in a UICollectionView that uses a `UICollectionViewDiffableDataSource`.

1. Initialize the data source
2. Set the header reference size
3. Register the custom header view as the collection view's `elementKindSectionHeader` supplementary view
4. Define the data source's `supplementaryViewProvider` closure
5. Apply your data to the data source

We can see these in action below.

{% highlight swift %}
final class ViewController: UIViewController {
    @IBOutlet private weak var collectionView: UICollectionView!
  
    // 1. Initialize the data source
    private lazy var dataSource = CollectionViewDataSource(collectionView)
    private lazy var collectionViewLayout: UICollectionViewFlowLayout = {
        let layout = UICollectionViewFlowLayout()
        // 2. Set the header reference size
        layout.headerReferenceSize = CGSize(width: collectionView.frame.width,
                                            height: 50)
        return layout
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        collectionView.collectionViewLayout = collectionViewLayout
        // 3. Register the header view
        collectionView.register(UINib(nibName: "HeaderCollectionReusableView",
                                      bundle: .main),
                                forSupplementaryViewOfKind: UICollectionView.elementKindSectionHeader,
                                withReuseIdentifier: "HeaderCollectionReusableViewReuseIdentifier")
        // 4. Define the `supplementaryViewProvider` closure
        dataSource.supplementaryViewProvider = { [weak self] collectionView, supplementaryViewKind, indexPath in
            switch supplementaryViewKind {
                case UICollectionView.elementKindSectionHeader:
                    guard let headerView = collectionView.dequeueReusableSupplementaryView(ofKind: supplementaryViewKind,
                                                                                           withReuseIdentifier: "HeaderCollectionReusableViewReuseIdentifier",
                                                                                           for: indexPath) as? HeaderCollectionReusableView
                    else {
                        fatalError("Failed to dequeue header view with reuse identifier `HeaderCollectionReusableViewReuseIdentifier`")
                    }
                    if let sectionIdentifier = self?.dataSource.snapshot().sectionIdentifiers[indexPath.section] {
                        // If you're using a view model, you can bind it to the header view here
                        headerView.setTitle("Section ID: \(sectionIdentifier.uuidString.prefix(5))")
                    }
                    return headerView
                default:
                    return nil
            }
        }
        // 5. Apply your data to the data source
        // If you're using a view model, you can use that to define the items below
        dataSource.apply([UUID(), UUID(), UUID(), UUID(), UUID()])
    }
}

/* -------------------------------------------------------------------------------
    Just added this part for completion.
    You can skip it if already familiar with `UICollectionViewDiffableDataSource`
*/
typealias SectionIdentifier = UUID
typealias ItemIdentifier = UUID

final class CollectionViewDataSource: UICollectionViewDiffableDataSource<SectionIdentifier, ItemIdentifier> {
  init(_ collectionView: UICollectionView) {
      super.init(collectionView: collectionView) { collectionView, indexPath, itemIdentifier in
          guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "CollectionViewCellReuseIdentifier",
                                                              for: indexPath) as? CollectionViewCell
          else {
              fatalError("Failed to dequeue cell with reuse identifier `CollectionViewCellReuseIdentifier`")
          }
          // If you're using a cell view model, you can bind it to the cell here
          cell.setTitle("Item ID:\n\(itemIdentifier.uuidString.prefix(5))")
          return cell
      }
  }

  func apply(_ sectionIdentifiers: [SectionIdentifier]) {
      var snapshot = NSDiffableDataSourceSnapshot<SectionIdentifier, ItemIdentifier>()
      snapshot.appendSections(sectionIdentifiers)
      sectionIdentifiers.forEach {
          // If you're using a view model, you can use that to define the items below
          snapshot.appendItems([UUID(), UUID(), UUID(), UUID()],
                                toSection: $0)
      }
      apply(snapshot,
            animatingDifferences: false)
  }
}
{% endhighlight %}

## Notes
- Unlike `UITableView` where we can implement `UITableViewDelegate` to define a custom header view regardless if we use `UITableViewDiffableDataSource` or `UITableViewDataSource`, it seems the same is not true for `UICollectionView`. When using `UICollectionViewDiffableDataSource`, we need to define the data source's `supplementaryViewProvider` closure instead.
- We need to set the header reference size, otherwise, the header will not appear as indicated by the [documentation](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout/1617710-headerreferencesize):

```
If the size in the appropriate scrolling dimension is 0, no header is added.

The default size values are (0, 0).
```

## Demo

Finally, here's a demo of the above. (Obviously I added some more UI code to the snippet above)

![Demo: Custom section header views in UICollectionView using UICollectionViewDiffableDataSource](/static_files/videos/2023-07-10-custom-section-header-uicollectionview-uicollectionviewdiffabledatasource_1.gif){: width="350" style="display: block; margin-left: auto; margin-right: auto;" }