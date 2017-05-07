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
CSV gehört nicht zur Grundausstattung von Sencha, es wird nor JSON, XML und Array unterstützt. Man braucht also einen neuen Reader, der den Import leistet.

### CsvReader

Vom Grundsatz her wird die *class*  _Ext.data.reader.Json_ mit neuen angepassten Methoden überschrieben. Die Magie steckt in der Methode _toJson()_ 

```javascript
Ext.define('CsvReader', {
    extend: 'Ext.data.reader.Json',
    alias : 'reader.csv',
    toJson: function(csvData){
      // Holla die Waldfeee
      return [];
    },
 
    // override, nett hier rein textuell, oder doch Teil des Compilers?
    getResponseData: function(response) {
        try {
            return this.readRecords(response.responseText);
        } catch (ex) {
            error = new Ext.data.ResultSet({
                total  : 0,
                count  : 0,
                records: [],
                success: false,
                message: ex.message
            });
            this.fireEvent('exception', this, response, error);
            return error;
        }
    },
 
    // override
    readRecords: function(strData) {
        var result = this.toJson(strData);
        return this.callParent([result]);
    }
});
```
Nun muss also neuer Code in das Projekt. Die Qualität dieses Readers kann nicht die Stabilität des Gesamtprojektes „runterziehen“. Beispielsweise ist die Lösung wie in [fahd.blog](http://fahdshariff.blogspot.de/2014/09/ext-js-csv-data-reader.html) sehr gefährlich, da sie viele Dinge unterstellt:
- [x] Datei ist im „richtigen“ Format,
- [x] Datei ist klein genug, um in den Speicher zu passen,
- [x] CSV ist genau in dem Format, also kommasepariert und in Doppelquote eingepackt,
- [x] unterstellt, dass in den Datensätzen keine Zeilenumbrüche sind.



## kundengerechte Darstellung eines Projektverlaufes 

