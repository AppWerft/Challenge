# Challenge

<img src="https://www.sencha.com/img/sencha-large.png" width="200" /> 

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

Es muss eine robuste, verlässliche Library her. Ohne das wirklich geprüft zu haben, könnte [Papa Parse](http://papaparse.com/) eine Lösung sein. 

Um so weit wie möglich in der Senchwelt zu bleiben, wird es sinnvolls ein, die Function:
```javascript
toJson: function(csvData){
      var result = Papa.parse(csvData[, config]);
      if (Array.isArrqay(result))
          return result;
    },  // eventuell Anbinddung an Error-System …

```
zu nutzen, weil man mit der String-Schnittstelle im Schema bleibt.  Die Konfiguration des Parsers ist [hier](http://papaparse.com/docs#config)

## kundengerechte Darstellung eines Projektverlaufes 

Für die Darstellung des Projektverlaufes bietet sich ein Gantt-Diagramm an. Außerhalb des Sencha-Welt hätte man die Auswahl in über 50 verschiedenen Chart-Libraries, beispielsweise [HighChart](https://www.highcharts.com/demo). Welche Lösung gewinnt hängt selbstverständlich von den Kundenanforderungen ab.

![](http://www.aceproject.com/img/ss/gantt-chart-en.jpg)

Highcharts unterstützt so wie Sencha von Haus aus keine Gantt-Diagramme. Das ist die Challenge ;-)



