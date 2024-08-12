qt，QListWidget

```css
QListView {
    outline: none;
    show-decoration-selected: 1;
}

QListView::item:alternate {
    background: #EEEEEE;
}

#listWidget::item {
    padding: 10px;
}

QListView::item:selected {
    border: 1px solid #6a6ea9;
}

QListView::item:selected:!active {
    background: qlineargradient(x1: 0, y1: 0, x2: 0, y2: 1,
                                stop: 0 #ABAFE5, stop: 1 #8588B2);
}

QListView::item:selected:active {
    background: qlineargradient(x1: 0, y1: 0, x2: 0, y2: 1,
                                stop: 0 #6a6ea9, stop: 1 #888dd9);
}

QListView::item:hover {
    background: qlineargradient(x1: 0, y1: 0, x2: 0, y2: 1,
                                stop: 0 #a1c6e3, stop: 1 #005ea8);
}
```

