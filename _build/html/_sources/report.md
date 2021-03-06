# Reports
## Allgemein

Die Nutzer des Dashboards haben ebenfalls die Möglichkeit sich zusätzlich zu unserem Dashboard weitere Informationen herunter zu laden. Zum Einen können sie sich noch einmal einen Report in Form eines PDF herunterladen. Zum Anderen besteht die Möglichkeit sich die Daten in Form einer Exceldatei herunter zu laden. Da das Ziel ist Menschen in Punkto Cyber Security zu sensibilisieren gibt es ebenfalls die Möglichkeit sich eine Powerpoint herunter zu laden, um andere über das Thema zu informieren.

### Generelles Vorgehen

Um die Möglichkeit zu bieten die Datein zu erstellen haben wir uns zunächst die Frage nach dem Zweck der Datei gestellt. Da unterschiedliche Bibliotheken verschiedene Vor- und Nachteile haben, konnten wir so besser einschätzen, welche wir nutzen wollen um das Ziel möglichst schnell zu erreichen. 

Wenn wir uns für eine Technologie entschieden haben, haben wir bei Problemen dennoch geschaut, wie es bei anderen Bibliotheken gemacht wird. Das hat uns besonders beim Erstellen des PDF Reports geholfen.

### Abbildung der Diagramme

Um downloadbare Dateien zur Verfügung zu stellen müssen die Diagramme zunächst in einer Form vorliegen, die von allen verwendeten Bibliotheken verwendet werden können. Zunächst suchten wir nach einer eleganten Art um die Datein zu verwalten. 

Es gab die Möglichkeit die erzeugten Bild-Datein, die wir aus den Diagrammen erstellen konnten in temporären Byte Buffern zu speichern. Dies hätte es ermöglicht die Datein nur kurzfristig im Arbeitsspeicher abzulegen und dann wieder zu löschen. Allerdings waren wir uns hier nicht sicher, dass alle Bibliotheken, die wir verwenden werden mit diesem Datenformat umgehen konnten. 

Deswegen entschieden wir uns die Daten als png-Dateien in dem Projektordner abzulegen. Auch wenn die Bilder hier dauerhaft liegen werden sie für jeden Download neu erstellt um immer die aktuellsten Diagramme zu beinhalten. 

Beispiel für die Erzeugung eines png Bildes:

```
pg = Phishing_Graphs() # Aufrufen der Klasse Phishing Graphs
# Erstellung des PNG eines Diagramms
pg.get_fail_bar('Abteilung', None, False).write_image("fail_bar_type_name.png", width=900, height=800, scale=1)

```

### Diagramm Anpassungen

Im Dashboard und in den Datein waren die Diagramme auch in unterschiedlichen visuellen Umgebungen. So war der Report beispielsweise mit einem weißen Hintergrund, während das Dashboard und die Powerpoint eher einen schwarzen Hintergrund haben. Deswegen haben wir bei fast allen Diagrammen den Darkmode eingeführt:

```

# Linien-Diagramm für Kosten erstellen 
def create_lineplot(self, year, darkmode=True): 
    if darkmode:
        color = "white"
        bg_color = "rgba(7, 37, 66, 0.8)"
    else:
        color = "black"  
        bg_color = "rgba(255, 255, 255, 0.8)"

    # Code bei der das Diagramm erstellt wird

    fig.update_layout(
        xaxis= dict(
                range=[self.df['year'].max() - 7 - 0.5, self.df['year'].max() + 0.5],
                dtick= 1,
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color, # hier wird die Tickfarbe je nach dem Darkmode bestimmt
                ticklen = 8,
                tickfont = dict(family = 'Arial', size = 14),
                showline = not darkmode, # Bei weißen Hintergründen wird so die Achsenlinie angezeigt
                linewidth = 1,
                linecolor = color, # hier wird die Linienfarbe je nach dem Darkmode bestimmt
        )
    )
```

## Daten in Excelform

Wenn die Nutzer nur die Daten verwenden wollen können sie diese auf jeder Seite des Dashboards runterladen. Die Daten werden dann in einer Exceldatei bereitgestellt

### Excel Datei mit einem Tabellenblatt

Um die Datei zu erstellen werden zunächst die Daten aus den vorhandenen Dataframes überarbeitet. Hier werden Überschriften angepasst und die unbenutzte Spalten entfernt. 

```
def get_excel_df(self):
        res = self.wc.df.copy()
        res.drop('size', inplace=True, axis=1)
        res.drop('note', inplace=True, axis=1)
        res = res.rename(
            columns={
                'Password': 'Passwort', 
                'category': 'Kategorie', 
                'rank': 'Platz', })
        return res.to_excel
```

Am Ende wird die Funktion des Dataframes to Excel zurückgegeben, die dann an das HTML-Element im Code zurückgegben wird und ausgeführt wird sobald der Nutzer den Button drückt. Ebenfalls werden dann der Name der Datei und der Name des Tabellenblattes festgelegt wird.

```
@app.callback(
    Output("download-password-excel", "data"),
    Input("password_btn", "n_clicks"),
    prevent_initial_call=True,
)
def download(n_clicks):
    return dcc.send_data_frame(pp.get_excel_df(), "Passwörter.xlsx", sheet_name="Passwörter")
```
### Excel Datei mit einem Tabellenblatt

Auf manchen Dashboard Seiten sind jedoch auch mehrere Datensätze hinterlegt. Um diese Daten trotzdem vorher alle in eine Exceldatei mit mehreren Tabellenblättern zu schreiben muss die Tabelle vorher mit einem Excelwriter geschrieben werden.

```
def get_excel(self):
    writer = pd.ExcelWriter('Phishing.xlsx', engine="xlsxwriter") # Name der Datei wird direkt am Anfang festgelegt
    copytoexcel = pd.DataFrame(self.pg.get_fail_df())
    copytoexcel.to_excel(writer, sheet_name="Fehlerquoten") # Ein Dataframe wird in ein Tabellenblatt der Excel Datei geschrieben
    copytoexcel = pd.DataFrame(self.pg.get_lia_df())
    copytoexcel.to_excel(writer, sheet_name="Phishing Absichten")
    writer.save()
```

Wenn der Nutzer nun die Datei anfordert wird die gespeicherte Excel Datei direkt an den Nutzer gesendet.

```
@app.callback(
    Output("download-phishing-excel", "data"),
    Input("phishing_btn", "n_clicks"),
    prevent_initial_call=True,
)
def download(n_clicks):
    phishing.get_excel()
    return dcc.send_file('Phishing.xlsx')
```

### Fazit zu Excel

Da sich Pandas Dataframes leicht in Excel Datein umwandeln lassen ist es sehr einfach gewesen die Daten dem Nuzter zur Verfügung zu stellen. Es mussten lediglich die Spaltennamen angepasst werden und in manchen Fällen die Datentypen so angepasst werden, dass Excel diese interpretieren konnte.

Das Ziel den Nutzern die Daten so zur Verfügung zu stellen konnte sehr einfach und schnell umgesetzt werden.

## Powerpoint

Es ist ebenfalls möglich die Diagramme und Informationen als Powerpoint zu exportieren. Das Ziel der Powerpoint ist es dem Nutzer eine Möglichkeit zu bieten, die Diagramme besser zu interpretieren. Gegebenenfalls kann die Präsentation auch von Zuhörern genutzt werden um andere für Themen der Cyber Sicherheit zu sensibilisieren.

Um mit Python Powerpoint Datein zu erstellen oder zu bearbeiten haben wir uns für die Bibliothek python-pptx verwendet. Diese Bibliothek scheint ziemlich verbreitet zu sein und wird häufig als Lösung im Internet präsentiert. 

### Erste Schritte

Die ersten Schritte um eine Powerpoint zu erstellen und zu speichern sind sehr einfach. Auch das Layout aus unserer Hauptpräsentation (von der Firma LaCTiS) zu übernehmen ging recht einfach, da es die Möglichkeit Templates zu laden und zu bearbeiten.

```
pp = Presentation("Template.pptx")

# Bearbeiten der Powerpoint

pp.save("Presentation.pptx")

```

### Layout

Die erste Herausforderung, die sich uns stellte, war das Layouten der einzelnen Folien. Unsere Diagramme hatten alle unterschiedliche Formate. So waren die Diagramme der Kosten eher im Querformat und die Diagramme aus Phishing hatten eher ein Hochformat.

Nachdem wir uns überlegt hatten, wie die einzelnen Folien ungefähr aussehen sollten ging es darum diese auch so mit python umzusetzen. Dazu hatten wir verschiedene Möglichkeiten. Zunächst haben wir versucht die Diagramme und Texte auf einer blanken Vorlagenfolie zu platzieren. Das stellte sich jedoch als sehr schwierig heraus, da man die Positionen auf der Folie mit Abständen zum Rand definiert. Für uns war es dann jedoch nicht möglich zu berechnen, wo der Text auf die Folie geschrieben werden sollte, denn wir hatten den Abstand zum Rand aber nicht die größe des Diagramms. Somit wurde es eher zu einem Trial-and-Error-Spiel die Texte richtig zu platzieren, das jedes mal von neuem losging, wenn Texte oder Diagramme angepasst wurden. **Das war also der falsche Weg.**

Deswegen fingen wir an mit Platzhaltern zu arbeiten. Wir haben also Masterfolien in unserem Template erstellt und die Bereiche an denen die Bilder eingefügt werden sollten entsprechend markiert. Später konnten wir dann in python auf die einzelnen PLatzhalter zugreifen und befüllen. 

Formatierungen von Texten und sonstige graphische Äunderungen haben wir ausschließlich in den Masterfolien von Powerpoint gemacht. Dies erschien am sinnvollsten und unkompliziertesten.

### Texte einfügen

Text konnte relativ einfach hinzugefügt werden:

```
slide = pp.slides.add_slide(self.pp.slide_layouts[2]) # Auswählen des Layouts für die Seite, die in die Präsentation eingefügt wird.

title = slide.placeholders[0]
title.text = "Was gibt es für Angriffsarten, die auf den Mensch abzielen?" # Titeltext verändern

sub_text = slide.placeholders[2]
tf = sub_text.text_frame # Textframe bekommen, in dem Text eingefügt werden kann
p = tf.add_paragraph() # Neuen Paragraphen für Text hinzufügen
p.text = 'Die größten Potentiale für Hacker sind die Komprimittierten oder Schwachen Anmeldedaten von Mitarbeitern und das Versenden von Phishing-Mails' # Paragraph mit Text befüllen
p.level = 0  

```

Mit einzelnen Paragraphen konnten einzelne Unterpunkte in das Textframe eingefügt werden. Die Formatierung haben wir wie bereits gesagt in den Masterfolien bereits hinterlegt.

### Diagramme einfügen

Bei den Diagrammen haben wir uns wiederum etwas schwer getan. Zwar konnten wir die Diagramme einfach in die PLatzhalter laden, jedoch wurden diese immer abgeschnitten, wenn sie nicht die Größe des Platzhalters hatten. Um das Problem zu lösen mussten wir, bei der Erzeugung der Bilder von den Diagrammen die Höhe und Breite festlegen. Dies konnte einfach als Variable an die Funktion write_image() übergeben werden, die die Plotly Express Figuren als Bilder abspeichert.

```
Phishing_Graphs().get_fail_bar('Branche', None, True).write_image("fail_bar_mark_pp.png", width=900, height=800)

```

Nachdem das festgelegt war mussten wir dann darauf achten, dass die Platzhalter in der Powerpoint Masterfolien dasselbe Seitenverhältniss hatten, damit nichts abgeschnitten wurde. Ebenfalls war es wichtig zu beachten wie groß die Höhe und Breite bei der Bilderzeugung angegeben wurde, denn wenn diese zu groß war, wurde die Beschriftung teilweise unleserlich.


### Fazit zu Powerpoint

Die Python-Bibliothek für Powerpoint funktioniert im Grunde recht gut. Einfach Präsentationen können damit relativ schnell umgesetzt werden. Wenn alles funktioniert und die Präsentation erstellt wird ist es auch sehr befriedigend zu sehen, wie die Änderungen der Diagramme automatisch in die Powerpoint übernommen werden ohne, dass man zusätzlichen Aufwand betreiben muss.

Der Weg dahin, dass alles funktioniert würden wir aber eher als mühselig beschreiben. Die Dokumentation der python-pptx Bibliothek ist eher dürftig und oft braucht es für Lösungen Workarounds, da Funktionen oft nicht einfach so funktionieren. Ein gutes Beispiel war hier die Erstellung der Folie über die Ziele von Phishing Mails:

```

added_img = 0
added_txt = 0

# Vorbereiten der Bilddateinamen und der Texte zu den Diagrammen
img_names = ["phishing_link_pp.png", "phishing_input_pp.png", "phishing_attach_pp.png"]
img_txt = [self.pg.get_text_for_dounut("link"), self.pg.get_text_for_dounut("input"), self.pg.get_text_for_dounut("attach")]

# itterieren über die Platzhalter der Folie, da die Indizes nicht angesteuert werden konnten
for plc in phishing_slide.placeholders:
    plc_type = str(plc.placeholder_format.type) # Öfteres aufrufen des Datentyps führt zu Fehlern. Deswegen wird es als Variable gespeichert.
    if "PICTURE" in plc_type:
        plc.insert_picture(img_names[added_img])
        added_img += 1 # Wenn ein Bild eingefügt wurde wird der Bildzähler erhöht
    if "OBJECT" in plc_type:
        tf = plc.text_frame
        p = tf.add_paragraph()
        p.text = img_txt[added_txt].replace("<br>", "") # Bezeichnungen aus dem Dashboard haben <br> Elemente, die nicht angezeigt werden sollen 
        added_txt += 1 # Wenn ein Text eingefügt wurde wird der Textzähler erhöht

```







