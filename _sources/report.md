## Powerpoint

Es ist ebenfalls möglich die Diagramme und Informationen als Powerpoint zu exportieren. Das Ziel der Powerpoint ist es dem Nutzer eine Möglichkeit zu bieten, die Diagramme besser zu interpretieren. Gegebenenfalls kann die Präsentation auch von Zuhörern genutzt werden um andere für Themen der Cyber Sicherheit zu sensibilisieren.

### Technisches Vorgehen

Um mit Python Powerpoint Datein zu erstellen oder zu bearbeiten haben wir uns für die Bibliothek python-pptx verwendet. Diese Bibliothek scheint ziemlich verbreitet zu sein und wird häufig als Lösung im Internet präsentiert. 

Die ersten Schritte um eine Powerpoint zu erstellen und zu speichern sind sehr einfach. Auch das Layout aus unserer Hauptpräsentation (von der Firma LaCTiS) zu übernehmen ging recht einfach, da es die Möglichkeit Templates zu laden und zu bearbeiten.

```
pp = Presentation("Template.pptx")

# Bearbeiten der Powerpoint

pp.save("Presentation.pptx")

```

Die erste Herausforderung, die sich uns stellte, war das Layouten der einzelnen Folien. Unsere Diagramme hatten alle unterschiedliche Formate. So waren die Diagramme der Kosten eher im Querformat und die Diagramme aus Phishing hatten eher ein Hochformat. 




