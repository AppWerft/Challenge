# Challenge

## Import einer CSV in Sencha extJS

Dateh werden intern in einem Model/Store gehalten. Für den Import externer Daten wird ein Proxy benötigt:
```javascript
store: new Ext.data.Store({
     model: 'myProject',
     proxy: {
       type: 'ajax',
       url: 'http://csv-uri/fake.txt',
       reader: new MyReader()
     },
}),
```
CSV gehört nicht zur Grundausstattung von Sencha, es wird nor JSON unD XML unterstützt. Man braucht aslo einen neuen Reader, der den Import leistet.


## kundengerechte Darstellung eines Projektverlaufes 

