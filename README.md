# PrenotaFarmaci - b2b API     ![logo g3m](https://www.g3m.it/wp-content/uploads/2016/10/g3_logo_top.png)

## Indice
* [Descrizione](#descrizione-progetto)
* [Lista endpoints](#lista-endpoints)
   1. [Token](#endpoint-per-la-richiesta-del-token)
   2. [Endpoints di test](#endpoints-di-test)
   3. [Endpoints di produzione](#endpoints-di-produzione)
* [Descrizione endpoints](#descrizione-endpoints)
* [Richieste](#richieste)
* [Risposte](#risposte)

## Descrizione progetto
Questo progetto è costituito dalle REST API che permettono di interfacciare qualsiasi client alla piattaforma PrenotaFarmaci. 
Le richieste e le risposte sono in formato **JSON**, l'autenticazione avviene tramite un'istanza di Oauth2 basata sul tipo di flow *Client Credentials Grant*.
Sono presenti degli endpoint di test identici agli endpoint di produzione, che permettono di facilitare lo sviluppo dei client.
Viene fornito su richiesta un progetto [SoapUi](https://www.soapui.org/) come supporto alla presente documentazione.
Il progetto è stato sviluppato utilizzando [Lumen](https://lumen.laravel.com/).
#### Sviluppi futuri
Fermo restando che verranno ascoltate e valutate le richieste dell'utenza in merito a qualsiasi miglioria e modifica, possiamo già indicare alcuni obiettivi di sviluppo a breve termine. 
* Pulizia generale del codice e correzione di eventuali bugs
* Aggiunta di funzioni *admin* per la gestione autonoma dei propri clients oauth2
* Aggiunta di ulteriori endpoints, pensati per gli utenti (siamo sempre in ambito b2b), che consentano operazioni di reportistica

## Lista Endpoints
### Endpoint per la richiesta del Token
  - [**/oauth/token**](#oauthtoken)
    <p>Permette il recupero del token oauth, il token ha durata di un anno</p>
### Endpoints di test
  - [**/fake-api/get-nre**](#get-nre)
    <p>Genera un Numero di Ricetta Elettronica sul server di test di SistemaTs</p>
  - [**/fake-api/get-farmacie**](#get-farmacie)
    <p>Ritorna un elenco di farmacie registrate sul server di test di SistemaTs</p>
  - [**/fake-api/check-cred**](#check-cred)
    <p>Controlla la validità delle credenziali della farmacia</p>
  - [**/fake-api/check-prom**](#check-prom)
    <p>Permette la visualizzazione del contenuto di un promemoria (senza pdf)</p>
  - [**/fake-api/lock-order**](#lock-order)
    <p>Effettua la presa in carico esclusiva dell'erogazione</p>
  - [**/fake-api/unlock-order**](#unlock-order)
    <p>Effettua la sospensione dell'erogazione</p>
  - [**/fake-api/get-prom**](#get-prom)
    <p>Permette di scaricare il promemoria in formato PDF</p>
### Endpoints di produzione

 **Per la descrizione fare riferimento agli endpoints di test**
    
  - **/api/check-cred**
    <p>Controlla la validità delle credenziali della farmacia</p>
  - **/api/check-prom**
    <p>Permette la visualizzazione del contenuto di un promemoria (*senza pdf*)</p>
  - **/api/lock-order**
    <p>Effettua la presa in carico esclusiva dell'erogazione</p>
  - **/api/unlock-order**
    <p>Effettua la sospensione dell'erogazione</p>
  - **/api/get-prom**
    <p>Permette di scaricare il promemoria in formato PDF</p><p></p>
    
## Descrizione endpoints

### oauth/token
  - Percorso: https://baseurl/oauth/token
  - Metodo: POST
  - Parametri: tipo di grant, id del client, api key
  - Tipo di risposta: JSON object
  - Parametri risposta: 'access_token'

  Esempio:
  ```php
  $guzzle = new GuzzleHttp\Client;

  $response = $guzzle->post('https://base.url/oauth/token', [
      'form_params' => [
         'grant_type' => 'client_credentials', //Costante
         'client_id' => 'client-id', 
          'client_secret' => 'client-secret',
          'scope' => '*', //Opzionale
      ],
  ]);

  return json_decode((string) $response->getBody(), true)['access_token']
  ```
### get-nre
  - Percorso: https://baseurl/fake-api/get-nre
  - Metodo: GET
  - Parametri: nessuno
  - Tipo di Risposta: JSON object
  - Parametri risposta: 'codFis', 'nre'
 
 Esempio:
 ```php
  $guzzle = new GuzzleHttp\Client(
    [
      'headers' => [Authorization' => 'Bearer ' . $oauthToken]
    ]
  );
  
  $response = $guzzle->get('https://base.url/fake-api/get-nre');
  
  $dati = json_decode($response->getBody(), true);
  
  $nre = $dati['nre'];
  $codFis = $dati['codFis'];
 
 ```
### get-farmacie
  - Percorso: https://baseurl/fake-api/get-farmacie
  - Metodo: GET
  - Parametri: nessuno
  - Tipo di Risposta: JSON object
  - Parametri risposta: array( 'regione_farmacia' => $datiFarmacia )
    ```json
    {
      "abruzzo":    {
        "codiceRegione": "000",
        "codiceAsl": "000",
        "codiceFarmacia": "12345",
        "pincode": "1234567890",
        "user": "U53RN4M3",
        "password": "P455W0RD"
      }
    }
 ```
 Esempio:
 ```php
  $guzzle = new GuzzleHttp\Client(
    [
      'headers' => [Authorization' => 'Bearer ' . $oauthToken]
    ]
  );
  
  $response = $guzzle->get('https://base.url/fake-api/get-farmacie');
  
  $dati = json_decode($response->getBody());
 ```
 ### check-cred
 - Percorso: https://baseurl/fake-api/check-cred
  - Metodo: POST
  - Parametri: [Dati Farmacia](#dati-farmacia)
  - Tipo di Risposta: JSON object
  - Parametri risposta: [Ricevuta Controllo Credenziali](#ricevuta-controllo-credenziali)<br/>
  Esempio:
  ```php
    $guzzle = new Client(
      [
        'headers' => [Authorization' => 'Bearer ' . $oauthToken]
      ]
    );
    
    $response = $guzzle->post('https://baseurl/fake-api/check-cred', [
      'form_params' => [
        'cod_reg'  => $formRequest->cod_reg,
        'cod_asl'  => $formRequest->cod_asl,
        'cod_uff'  => $formRequest->cod_uff,
        'pin'      => $formRequest->pin,
        'user'     => $formRequest->user,
        'password' => $formRequest->password
      ]
    ]);
    
    $ricevutaCredenziali = json_decode($response->getBody());
  ```
### check-prom
  - Percorso: https://baseurl/fake-api/check-prom
  - Metodo: POST
  - Parametri: [Dati Farmacia](#dati-farmacia), [Dati Ricetta](#dati-ricetta)
  - Tipo di Risposta: JSON object
  - Parametri risposta: [Ricevuta Visualizzazione](#ricevuta-visualizzazione)<br/>
  
  Questa chiamata effettua 2 differenti operazioni: **presa in carico promemoria**, **sospensione erogazione**
    <br/> questo consente di visualizzare i dati contenuti nel promemoria senza vincolarne l'erogazione alla farmacia.<br/>
  Esempio:
  ```php
    $guzzle = new Client(
      [
        'headers' => [Authorization' => 'Bearer ' . $oauthToken]
      ]
    );
    
    $response = $guzzle->post('https://baseurl/fake-api/check-prom', [
      'form_params' => [
        'cod_reg'  => $formRequest->cod_reg,
        'cod_asl'  => $formRequest->cod_asl,
        'cod_uff'  => $formRequest->cod_uff,
        'pin'      => $formRequest->pin,
        'user'     => $formRequest->user,
        'password' => $formRequest->password
        'nre'      => $formRequest->nre,
        'cod_fis'  => $formRequest->codFis
      ]
    ]);
    
    $ricevutaVisualizzazione = json_decode($response->getBody());
  ```
### lock-order
   - Percorso: https://baseurl/fake-api/lock-order
  - Metodo: POST
  - Parametri: [Dati Farmacia](#dati-farmacia), [Dati Ricetta](#dati-ricetta)
  - Tipo di Risposta: JSON object
  - Parametri risposta: [Ricevuta Visualizzazione](#ricevuta-visualizzazione)<br/>
  
  Questa chiamata consente la presa in carico esclusiva del promemoria (la farmacia viene vincolata all'erogazione)<br/>
  
  Nella [ricevuta](#ricevuta-visualizzazione) sarà presente una [comunicazione](#comunicazione) con *codice == 200* e *message == 'Presa in carico esclusiva completata'* <br/>
  Esempio:
  ```php
    $guzzle = new Client(
      [
        'headers' => [Authorization' => 'Bearer ' . $oauthToken]
      ]
    );
    
    $response = $guzzle->post('https://baseurl/fake-api/lock-order', [
      'form_params' => [
        'cod_reg'  => $formRequest->cod_reg,
        'cod_asl'  => $formRequest->cod_asl,
        'cod_uff'  => $formRequest->cod_uff,
        'pin'      => $formRequest->pin,
        'user'     => $formRequest->user,
        'password' => $formRequest->password
        'nre'      => $formRequest->nre,
        'cod_fis'  => $formRequest->codFis
      ]
    ]);
    
    $ricevutaVisualizzazione = json_decode($response->getBody());
  ```
  ### unlock-order
   - Percorso: https://baseurl/fake-api/unlock-order
  - Metodo: POST
  - Parametri: [Dati Farmacia](#dati-farmacia), [Dati Ricetta](#dati-ricetta)
  - Tipo di Risposta: JSON object
  - Parametri risposta: [Ricevuta Visualizzazione](#ricevuta-visualizzazione)<br/>
  
  Questa chiamata consente la sospensione dell'erogazione del promemoria (qualunque farmacia potrà erogare questo promemoria)<br/>
  
  Nella [ricevuta](#ricevuta-visualizzazione) sarà presente una [comunicazione](#comunicazione) con *codice == 0000* e *message == 'Revoca della ricetta avvenuta con successo'* <br/>
  Esempio:
  ```php
    $guzzle = new Client(
      [
        'headers' => [Authorization' => 'Bearer ' . $oauthToken]
      ]
    );
    
    $response = $guzzle->post('https://baseurl/fake-api/unlock-order', [
      'form_params' => [
        'cod_reg'  => $formRequest->cod_reg,
        'cod_asl'  => $formRequest->cod_asl,
        'cod_uff'  => $formRequest->cod_uff,
        'pin'      => $formRequest->pin,
        'user'     => $formRequest->user,
        'password' => $formRequest->password
        'nre'      => $formRequest->nre,
        'cod_fis'  => $formRequest->codFis
      ]
    ]);
    
    $ricevutaVisualizzazione = json_decode($response->getBody());
  ```
### get-prom
  - Percorso: https://baseurl/fake-api/get-prom
  - Metodo: POST
  - Parametri: [Dati Farmacia](#dati-farmacia), [Dati Ricetta](#dati-ricetta)
  - Tipo di Risposta: JSON object
  - Parametri risposta: [Ricevuta PDF](#ricevuta-pdf)<br/>
  
  Questa chiamata effettua 2 operazioni: **presa in carico esclusiva** (se già avvenuta si trasforma in semplice visualizzazione)   e **generazione di promemoria PDF** secondo il modello ministeriale <br/>
  
  Esempio:
  ```php
    $guzzle = new Client(
      [
        'headers' => [Authorization' => 'Bearer ' . $oauthToken]
      ]
    );
    
    $response = $guzzle->post('https://baseurl/fake-api/get-prom', [
      'form_params' => [
        'cod_reg'  => $formRequest->cod_reg,
        'cod_asl'  => $formRequest->cod_asl,
        'cod_uff'  => $formRequest->cod_uff,
        'pin'      => $formRequest->pin,
        'user'     => $formRequest->user,
        'password' => $formRequest->password
        'nre'      => $formRequest->nre,
        'cod_fis'  => $formRequest->codFis
      ]
    ]);
    
    $ricevutaPDF = json_decode($response->getBody());
  ```
## Richieste

### Dati farmacia
    
   - **cod_reg**  -> codice regione farmacia (3 cifre) es. *130*
   - **cod_asl**  -> codice ASL farmacia (3 cifre) es. *204*
   - **cod_uff**  -> codice farmacia assegnato da ASL (5 cifre) es. *12345*, oppure *00123*
   - **pin**      -> pin della farmacia, assegnato da Sistema ts (10 cifre) es. *1234567890*
   - **user**     -> username di sistemaTs della farmacia
   - **password** -> password di sistemaTs della farmacia
   
### Dati ricetta
   - **nre**     -> Numero di Ricetta Elettronica (15 caratteri) es. *1300A1234567890*
   - **cod_fis** -> Codice fiscale dell'intestatario del promemoria (16 caratteri alfanumerici) es. *RSSMRA80G01F478R*
   
## Risposte
  
### Ricevuta Visualizzazione
Questa è la ricevuta di base, tutte le altre ricevute estendono questa classe aggiungendo alcuni parametri specifici. Tutti i parametri che non vengono utilizzati nelle ricevute spcifiche sono settati a *null*.
  
  Parametri:
   - **nome_cognome**        -> nome e cognome dell'intestatario del promemoria
   - **cod_fiscale**         -> codice fiscale dell'intestatario del promemoria
   - **nre**                 -> Numero di Ricetta Elettronica associato al promemoria
   - **medico**              -> Nome e cognome del medico prescrittore
   - **esenzione**           -> Codice dell'esenzione o *Non esente*
   - **prescrizioni**        ->Elenco delle [prescrizioni](#prescrizione) presenti nel promemoria
   - **risultatiOperazioni** -> Array di valori di controllo della correttezza delle operazioni
   - **error**               -> *true* se la chiamata ha prodotto un errore, altrimenti *false*
   - **errorBag**            -> *null* se *error == false* altrimenti contiene un array di [errori](#errore)
   - **comunicationsBag**    -> *null*, oppure contiene un array di [comunicazioni](#comunicazione)
   ```json
  {
   "nome_cognome": "MARIO PINO",
   "cod_fiscale": "PNIMRA70A01H501P",
   "nre": "1300A4004599125",
   "medico": "PRO VA",
   "esenzione": "Non esente",
   "prescrizioni": {"farmaci":    [
            {
         "nome_farmaco": "AULIN 100MG 30BS",
         "codice_aic": "025940053",
         "cod_gruppo_eq": "32A",
         "desc_gruppo_eq": "NIMESULIDE 100MG 30 UNITA' USO ORALE",
         "nota": "066",
         "quantity": "1"
      },
            {
         "nome_farmaco": "ACARPHAGE 100MG 40CPR BL",
         "codice_aic": "038835144",
         "cod_gruppo_eq": "H1A",
         "desc_gruppo_eq": "ACARBOSIO 100MG 40 UNITA' USO ORALE",
         "nota": "nessuna nota",
         "quantity": "1"
      }
   ]},
   "risultatiOperazioni": [1],
   "error": false,
   "errorBag": null,
   "comunicationsBag": null
}
```
   
### Ricevuta controllo credenziali
Estende [*ricevuta visualizzazione*](#ricevuta-visualizzazione) e aggiunge un singolo campo
- **checkResults** -> *true* se le credenziali sono valide, altrimenti *false*.<br/>Se *checkResults* è *false*, allora *error == true* e *errorBag* contiene gli [errori](#errore) corrispondenti. 
   
### Prescrizione
  - **nome_farmaco**   -> nome commerciale del farmaco come inserito in promemoria
  - **codice_aic**     -> codice identificativo del farmaco (univoco)
  - **cod_gruppo_eq**  -> codice del gruppo di equivalenza del farmaco
  - **desc_gruppo_eq** -> descrizione del gruppo di equivalenza del farmaco
  - **nota**           -> nota del farmaco se presente (forma a 3 cifre es. 001, 066)
  - **quantity**       -> quantità di confezioni prescritte
  
  ```json
  {
         "nome_farmaco": "ACARPHAGE 100MG 40CPR BL",
         "codice_aic": "038835144",
         "cod_gruppo_eq": "H1A",
         "desc_gruppo_eq": "ACARBOSIO 100MG 40 UNITA' USO ORALE",
         "nota": "nessuna nota",
         "quantity": "1"
      }
  ```
### Ricevuta PDF
  Estende [*ricevuta visualizzazione*](#ricevuta-visualizzazione) e aggiunge alcuni campi.
  
  - **rawPayload** -> campo utilizzato in fase di debug dallo sviluppatore (verrà rimosso in futura versione)
  - **promPayload** -> campo utilizzato in fase di debug dallo sviluppatore (verrà rimosso in futura versione)
  - **PDF** -> contiene la string base64 che rappresenta il PDF del promemoria
  
  Esempio di conversione:
  ```php
    $b64str = "JVBERi0xLjMKMSAw...PgpzdGFydHhyZWYKNTY1MAolJUVPRgo=";
    file_put_contents('mypdf.pdf', base64_decode($b64str));
    
  ```
  
### Errore
  - **code**    -> codice errore
  - **message** -> messaggio errore
  
  ```json
  {
    "errorBag": {"bag": 
    [ 
      {
        "code": 5064,
        "message": "Dati erogatore (Codice regione, Codice ASL, Codice SSA) non validi"
       }
     ]
  }
  ```

### Comunicazione
  - **code**    -> codice comunicazione
  - **message** -> messaggio comunicazione
  
  ```json
  {
    "comunicationsBag": {"comunicazioni": 
    [ 
      {
        "code": "0000",
        "message": "Revoca della ricetta avvenuta con successo"
       }
     ]
  }
  ```
  
  
  
