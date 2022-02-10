# Dashboard

## Genereller Aufbau

Das Dashboard besteht aus mehreren Tabs, die unterschiedliche Themen behandeln. Dabei gehen wir Thematisch vor. Zunächst kann man sich im Tab "Schaden durch Hacks" einen ersten Überblick verschaffen, was für Schäden Cyber Attacken im Allgemeinen nach sich ziehen. Um sich weiter zu informieren wird im Tab "Hackermethoden" genauer darauf eingegangen, wie Hacks durchgeführt werden und welche Arten am meisten verbreitet sind. Die zweit häufigsten Angriffe, mit der Schwachstelle Mensch sind Phishing Attacken. Deswegen kann sich der Nutzer im Tab "Phishing" über mögliche Absichten von solchen Mails informieren und auch schauen, welche Branchen und Abteilungen häufig betroffen sind. Im letzten Tab "Passwortsicherheit" wird, wie der Name schon sagt, dann noch auf die Passwortsicherheit ein. Hier kann der Nutzer Passörter testen und schauen, ob sich vielleicht eins seiner verwendeten Passwörter in der Wordcloud der meist verwendeten Passwörter wiederfindet.

## Verwendete Technologien

### Applikation

Das Dashboard haben wir mit Dash umgesetzt. Die Applikation zu erstellen und zu starten war dann relativ einfach:

```
import dash
app = dash.Dash(__name__)


if __name__ == '__main__':
    app.run_server(debug=True)

```

Für die Gestaltung des Dashboard haben wir eine app.css Datei erstellt, die wir im Ordner assets hinterlegt haben. Hier wurden Sachen definiert wie beispielsweise die Hintergrundfarbe sowie das Aussehen der Überschriften:

```
body {
    font-family: sans-serif;
    background-color: #072542;
    color: #FFFFFF;
}
h1, h2, h3, h4, h5, h6 {
    color: #FFFFFF;
    text-align: center;
}
```

### Datenhandling

Die Daten haben, die wir verwenden, liegen uns zu Beginn in Excel vor. Mit dem Framework Pandas lesen wir diese Daten ein und bearbeiten sie. Pandas verwenden wir als zentrales Tool um Daten zu manipulieren und auch um Daten am Ende für den Export bereit zu stellen. 

```
self.df = pd.read_excel("./datasets/DataBreaches.xlsx")

self.df = self.df.rename(columns= {'year   ':'year'}) # Spalte year umbennenen (aufgrund von Leerzeichen nach "year")
self.df = self.df.drop(self.df.index[[0]]) # Löschen der ersten Zeile
self.df["year"] = self.df["year"].astype("int") # Umwandlung des Datentyps von Year von Object in Integer mit der typecasting_from_column-Methode
self.df["records lost"] = pd.to_numeric(self.df["records lost"]) # Umwandlung des Datentyps von records lost von Object in numeric
self.df["organisation"] = self.df["organisation"].astype("string") # Umwandlung des Datentyps von Organisation von Object in string
self.df['organisation_name'] = ''
```

Das Dataframe kann dann in der ganzen Klasse verwendet werden um Diagramme zu erstellen oder zum Download bereit gestellt werden.

Das Framework ist dabei sehr tiefgehend und bietet viele Funktionen, die wir leider nicht alle verstehen oder noch nicht richtig anwenden können. Deswegen mussten wir auch häufig Workarounds für bestimmte Probleme finden. Zum Beispiel wollten wir einmal die Zeitpunkt von Databreaches gleichmäßig im Monat verteilen, sodass im Scatter-Plot-Diagramm die Daten nicht übereinander liegen sondern sich gelichmäßig verteilen:

```
for year in range( 2004, 2022):
    # Herausfiltern der Firmennamen, bei denen die Schaden zu den größten 10% gehören, damit kleinere Punkte keine Beschriftung haben
    self.df['organisation_name'] = np.where((self.df['year'] == year) & (self.df['records lost'] >= self.df['records lost'].max()*0.1), self.df['organisation'], self.df['organisation_name'])
    
    # gleichmäßiges Verteilen der einzelnen Vorfälle auf einen Monat
    for month in range(1, 13):
        date = str(year) + '-' + str(month).zfill(2) + '-01'

        new_df = self.df.loc[(self.df["date"] == date)]

        step = 30//(new_df.shape[0]+1) # Stepgröße um zu berechnen, in welchem Abstand die Vorfälle platziert werden
        start = 0
        for index, row in new_df.iterrows():
            self.df.loc[index, 'date'] = '%s-%s-%s' % (year, str(month).zfill(2), str(start+step).zfill(2))
            start += step
```

### Diagramme

Die Diagramme haben wir zum großen Teil mit Plotly Express erstellt. Die Diagramme konnten dadurch im ersten Schritt schnell erstellt werden aus den vorhandenen Daten. 

```
fig = px.line(
    self.df_fig1, 
    x="year", 
    y="records lost", 
    labels={
        "year": "",
        "records lost": "entstandener Schaden (in Mrd. US$)",
    },
    title='', 
    markers=True
)

```
Die Anpassung und der feinschliff war dafür leider etwas schwieriger und teilweise konnten wir bestimmte Vorhaben auch nicht umsetzen. 

```
fig.update_xaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_yaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_layout(

            # Anpassen des Titels
            title={
                'text': "Verlauf des enstandenen Schadens durch Datenlecks",
                # Titel linksbündig an die y-Achse anpassen
                'y':0.87, # Fehleranfällig, da die Größe des Diagramms über die Position der Überschrift bestimmt
                'x':0.0,
                'xref': "paper",
                'xanchor': 'left',
                'yanchor': 'top'
            },

            # Hier wird der Hintergrund transparent gemacht und die Schriftfarbe je nach Darkmoder bestimmt (siehe Report/Allgemein/Diagramm Anpassungen)
            plot_bgcolor = "rgba(0,0,0,0)",
            paper_bgcolor = "rgba(0,0,0,0)",
            font_color=color,

            # Anpassen der X- und Y-Achse
            xaxis= dict(
                range=[self.df['year'].max() - 7 - 0.5, self.df['year'].max() + 0.5],
                dtick= 1,
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                tickfont = dict(family = 'Arial', size = 14),
                showline = not darkmode,
                linewidth = 1,
                linecolor = color,
            ),

            yaxis = dict(
                range=[0, df_fig1['records lost'].max()*1.2],
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                ticklabelposition="outside",
                showline = True,
                linewidth = 1,
                linecolor = color,
            ),
        )
        
        # Anpassen der Farbe der Linien
        fig.update_traces(
            marker = dict(
                color = '#4DDBE3',
                size = 10,
                opacity = 0.8,
            ),
            line = dict(
                color = '#4DDBE3',
                width = 2
            ),
        )
```

Auf die einzelnen Diagramme werden wir aber später auch noch genauer eingehen


## Ordnerstruktur und Technischer Aufbau

Die Struktur haben wir so aufgebaut, dass wir für jeden Tab eine eigene Klasse erstellt haben. Jede Klasse bietet dabei eine get_layout() Funktion, die das Dash Layout zurückgibt. Durch das aufteilen konnte sowohl Hauptdatei deutlich übersichtlicher gestaltet werden und man wusste direkt, wo man suchen sollte, wenn ein Fehler auftritt. Wir haben uns auch dafür entschieden die einzelnen Diagramme für die Seiten ebenfalls in eigenen Klassen zu verwalten. Dadurch wurde die Logik von Diagramm erstellen und Dashboard sauber voneinander getrennt.


### dashboard.py / Einstiegspunkt

Der Einstiegspunkt der Appliktaion bietet die "dashboard.py" Datei. Hier wird die Dash-App erzeugt und mit dem grundlegendem Layout sowie allen Funktionen die im Dashboard gebraucht werden erzeugt. Im Layout legen wir zunächst nur das Aussehen des Headers fest.

```

app.layout = html.Div(style={'backroudColor': 'green'}, children=[
    html.Div([
        html.Header(children=[
            html.Div(
                id="app-header",
                children=[
                    # Festlegen des Titels

                    html.H1(
                        id="app-title",
                        children='Welcome to LaCTiS',
                        style={
                            
                        }
                    ),

                    # Abschnitt der einzlenen Tabs
                    dcc.Tabs(
                        id="tabs-container", 
                        value='tab_databreaches', 
                        parent_className='custom-tabs',

                        children=[
                            dcc.Tab(
                                className="custom-tab", 
                                label='Schaden durch Hacks', 
                                value='tab_databreaches',
                                selected_className='custom-tab--selected'
                            ),
                            dcc.Tab(
                                className="custom-tab", 
                                label='Hackermethoden', 
                                value='tab_methods',
                                selected_className='custom-tab--selected'
                            ),
                            dcc.Tab(
                                className="custom-tab", 
                                label='Phishing', 
                                value='tab_phishing',
                                selected_className='custom-tab--selected'
                            ),
                            dcc.Tab(
                                className="custom-tab", 
                                label='Passwortsicherheit', 
                                value='tab_password',
                                selected_className='custom-tab--selected'
                            ),
                        ]
                    ),  
                ]
            ),
        ]),
        html.Br(),html.Br(),html.Br(),
        
        # Hier wird der Inhalt dynamisch je nach Tabauswahl erzeugt
        html.Div(id="tabs-content")
    ]),
     
])
```

Die Tabauswahl beeinflusst dann, welches Layout dann als Inhalt der Seite angezeigt werden:

```
@app.callback(Output('tabs-content', 'children'),
              Input('tabs-container', 'value'))
def render_content(tab):
    if tab == 'tab_password':
        return pp.get_layout()
    elif tab == 'tab_methods':
        return dbav.get_layout()
    elif tab == 'tab_databreaches':
        return dbp.get_layout()
    elif tab == 'tab_phishing':
        return phishing.get_layout()
```

### data_breches_cost.py / Schaden durch Hacks

Hier wollen wir dem Nutzer klar machen, was für Schaden Cyber Attacken anrichten können. Der Anwender kann sehen, wie sich die Summe der einzelnen Jahre, sowie der Durchschnitt verhält. Ebenfalls wird dargestellt, was die größten Schäden durch Datenlecks bei einzelnen Firmen in einem Jahr waren. Dem Nutzer wird außerdem ein Slider angeboten, mit dem er verschiedene Jahre auswählen kann.

#### Summierte Schäden

Es werden für die einzelnen Jahre die Schäden summiert und dargestellt. Um dem Nutzer ein Gefühl zu geben, ob es eher zunimmt oder abnimmt haben wir ebenfalls den Durchschnitt berechnet.
Das Diagramm besteht dabei eigentlich aus vier Diagrammen: 

**Summe der Einzelnen Jahre**
```
sum_df = pd.DataFrame(self.df.groupby(by=['year'])['records lost'].sum()/1000).reset_index()

df_fig1 = (sum_df)
        df_fig1 = df_fig1.loc[df_fig1["year"]>= 2014]
        # print(df_fig1.head())
        fig = px.line(df_fig1, x="year", y="records lost", 
                                labels={
                                "year": "",
                                "records lost": "entstandener Schaden (in Mrd. US$)",

                                },
                                

                                title='', markers=True)

        fig.update_xaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_yaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_layout(
            title={
                'text': "Verlauf des enstandenen Schadens durch Datenlecks",
                'y':0.87,
                'x':0.0,
                'xref': "paper",
                'xanchor': 'left',
                'yanchor': 'top'},
            plot_bgcolor = "rgba(0,0,0,0)",
            paper_bgcolor = "rgba(0,0,0,0)",
            font_color=color,

        
            xaxis= dict(
                range=[self.df['year'].max() - 7 - 0.5, self.df['year'].max() + 0.5],
                dtick= 1,
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                tickfont = dict(family = 'Arial', size = 14),
                showline = not darkmode,
                linewidth = 1,
                linecolor = color,
                
                

                
                ),
            yaxis = dict(
                range=[0, df_fig1['records lost'].max()*1.2],
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                ticklabelposition="outside",
                showline = True,
                linewidth = 1,
                linecolor = color,
                
                ),
            )
        

        fig.update_traces(
            marker = dict(
                color = '#4DDBE3',
                size = 10,
                opacity = 0.8,
            ),
            line = dict(
                color = '#4DDBE3',
                width = 2
            ),
        )

        avg_fig = px.line(avg_year, x="year", y="avg", 
                                title='Testtitle', markers=False, line_shape='spline')
        avg_fig.update_traces(
           
            line = dict(
                smoothing=0.8,
                color = 'rgb(159, 90, 253)',
                width = 4
            ),
        )
```
- Durchschnitt über die Jahre 
- Punktmarker welches Jahr markiert ist für die Jahressumme (Scatterplot)
- Punktmarker welches Jahr markiert ist für den Durchschnitt (Scatterplot)
