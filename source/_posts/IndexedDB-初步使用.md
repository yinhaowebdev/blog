title: IndexedDB 初步使用
author: Billy Yin
tags: []
categories: []
date: 2022-03-09 18:09:00
---
```js
// 定义类
export default class CacheManager {
  constructor(dbName = 'name', tableName = 'table') {
    this.init(dbName, tableName);
  }

  init(dbName = 'iotServer', tableName = 'pageData') {
    return new Promise((resolve, reject) => {
      this.indexedDB =
        window.indexedDB ||
        window.webkitIndexedDB ||
        window.mozIndexedDB ||
        window.msIndexedDB;
      this.IDBOpenDBRequest = this.indexedDB.open(dbName);
      this.db = undefined;
      this.tableName = tableName;
      this.IDBOpenDBRequest.onsuccess = (e) => {
        this.db = e.target.result;
        resolve();
      };
      this.IDBOpenDBRequest.onupgradeneeded = (e) => {
        this.db = e.target.result;
        if (!this.db.objectStoreNames.contains(this.tableName)) {
          // objectStore就相当于数据库中的一张表。IDBObjectStore类型。
          this.objectStore = this.db.createObjectStore(this.tableName, {
            keyPath: 'id',
            autoIncrement: false
          });
        }
        resolve();
      };
    });
  }

  addValue(key, value) {
    return new Promise((resolve, reject) => {
      const request = this.db?.transaction([this.tableName], 'readwrite').objectStore(this.tableName).add({
        id: key,
        ...value
      });
      request.onsuccess = (e) => {
        resolve(e.target.result);
      };
      request.onerror = (e) => {
        reject(e.target.error);
      };
    });
  }

  updateValue(key, value) {
    return new Promise((resolve, reject) => {
      const request = this.db?.transaction([this.tableName], 'readwrite').objectStore(this.tableName).put({
        id: key,
        ...value
      });
      request.onsuccess = (e) => {
        resolve();
      };
      request.onerror = (e) => {
        reject(e);
      };
    });
  }

  getValue(key) {
    return new Promise((resolve, reject) => {
      const request = this.db?.transaction([this.tableName], 'readonly').objectStore(this.tableName).get(key);
      request.onsuccess = (e) => {
        resolve(request.result);
      };
      request.onerror = (e) => {
        reject(e);
      };
    });
  }
}
```

```js
// 使用
let cacheManager = new CacheManager()
await cacheManager.init()
await cacheManager.addValue(1, {name: 'alen'})
await cacheManager.getValue(1) // {id:1, name: 'alen'}
```
