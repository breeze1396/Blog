qt，QListWidget，居中

```c++
 QListWidget* listView = ui.listWidget;
 listView->setItemSelected(listView->item(0), true);
 
 //遍历所有item，设置居中
 for (int i = 0; i < listView->count(); i++)
 {
     QListWidgetItem* item = listView->item(i);
     item->setTextAlignment(Qt::AlignCenter);
 }
```

![image-20240723161749684](./listWedget居中.assets/image-20240723161749684.png)