# Reports
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

Nachdem wir uns überlegt hatten, wie die einzelnen Folien ungefähr aussehen sollten ging es darum diese auch so mit python umzusetzen. Dazu hatten wir verschiedene Möglichkeiten. Zunächst haben wir versucht die Diagramme und Texte auf einer blanken Vorlagenfolie zu platzieren. Das stellte sich jedoch als sehr schwierig heraus, da man die Positionen auf der Folie mit Abständen zum Rand definiert. Für uns war es dann jedoch nicht möglich zu berechnen, wo der Text auf die Folie geschrieben werden sollte, denn wir hatten den Abstand zum Rand aber nicht die größe des Diagramms. Somit wurde es eher zu einem Try-and-Error-Spiel die Texte richtig zu platzieren, das jedes mal von neuem losging, wenn Texte oder Diagramme angepasst wurden. **Das war also der falsche Weg.**

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

Der Weg dahin, dass alles funktioniert würden wir aber eher als mühselig beschreiben. Die Dokumentation der python-pptx Bibliothek ist eher dürftig und oft braucht es für Lösungen Workarounds, da oft Dinge nicht einfach so funktionieren. Ein Beispiel war hier die Folie über die Ziele von Phishing Mails:

```

added_img = 0
added_txt = 0

# Vorbereiten der Bilddateinamen und der Texte zu den Diagrammen
img_names = ["phishing_link_pp.png", "phishing_input_pp.png", "phishing_attach_pp.png"]
img_txt = [self.pg.get_text_for_dounut("link"), self.pg.get_text_for_dounut("input"), self.pg.get_text_for_dounut("attach")]

# itterieren über die Platzhalter der Folie, da die Indizes nicht angesteuert werden können
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




