# Vanhan kalustearvio projektin esimerkki Node.js koodina

Tämä projekti on Node.js-pohjainen sovellus, joka käyttää Google Gemini AI -mallia huonekalujen hinta-arvioiden tekemiseen ja kuvien analysointiin.

## Ominaisuudet

1. Huonekalujen tunnistus ja analysointi kuvista
2. Kuvan optimointi ennen analysointia
3. Strukturoidun datan tuottaminen Zod-skeeman avulla
4. OpenAI:n GPT-4 Vision -mallin hyödyntäminen kuva-analyysissa
5. Gemini AI -mallin hyödyntäminen tietojen käsittelyssä

## Tiedostorakenne

- `priceAnalysis.js`: Sisältää logiikan hinta-arvioiden tekemiseen Gemini AI:n avulla
- `ZodExample.js`: Sisältää logiikan kuvien analysointiin ja datan strukturointiin
- `imageAnalysis.js`: Sisältää logiikan kuvien analysointiin (ei käytössä tässä versiossa)
- `index.js`: Pääsovellustiedosto, joka demonstroi hinta-arvioinnin toimintaa

## Asennus

1. Kloonaa tämä repositorio
2. Asenna riippuvuudet komennolla `npm install`
3. Kopioi `.env.example` tiedosto nimellä `.env`:
   ```
   cp .env.example .env
   ```
4. Avaa `.env` tiedosto ja lisää sinne oma GEMINI_API_KEY

## Käyttö

Aja sovellus komennolla:

```
npm start
```

Tämä suorittaa `index.js`-tiedoston, joka demonstroi hinta-arvioinnin toimintaa kovakoodatulla esimerkillä.

## Toiminnallisuuden kuvaus

1. `priceAnalysis.js`:

   - `estimateFurniturePrice`-funktio ottaa vastaan huonekalun tiedot ja lähettää ne Gemini AI -mallille analysoitavaksi
   - Gemini AI palauttaa hinta-arvion ja muita tietoja JSON-muodossa

2. `imageAnalysis.js`:

   - `analyzeImage`-funktio on suunniteltu analysoimaan kuvia Gemini AI:n avulla
   - Tämä toiminnallisuus ei ole vielä käytössä nykyisessä versiossa

3. `index.js`:
   - Demonstroi hinta-arvioinnin toimintaa kovakoodatulla esimerkillä
   - Tulostaa huonekalun tiedot ja saadun hinta-arvion konsoliin

4. `ZodExample.js`:

- `analyzeFurnitureWithZod`-funktio ottaa vastaan kuvatiedoston polun
- Kuva optimoidaan Sharp-kirjaston avulla
- Optimoitu kuva lähetetään OpenAI:n GPT-4 Vision -mallille analysoitavaksi
- Analyysin tulos validoidaan ja muotoillaan Zod-skeeman avulla
- Funktio palauttaa strukturoidun objektin, joka sisältää tiedot analysoidusta huonekalusta

Zod-skeema (`FurnitureAnalysis`) määrittelee seuraavat kentät:

- type: Huonekalun tyyppi
- brand: Valmistaja tai suunnittelija
- model: Mallinimi tai -numero
- color: Pääväri
- dimensions: Mitat (pituus, leveys, korkeus)
- age: Arvioitu ikä vuosissa
- condition: Kunto
- haagaHelia: Onko huonekalu Haaga-Heliasta (boolean) ( Tämä on lisätty vain esimerkin vuoksi )

## Huomioitavaa

- Tämä on prototyyppiversio, joka demonstroi toiminnallisuutta
- Todellisessa sovelluksessa tiedot tulisivat käyttöliittymästä tai API-kutsusta
- Promptia ja mallin asetuksia voi säätää tarpeen mukaan paremman tarkkuuden saavuttamiseksi
- TypeScriptin käyttö parantaisi koodin laatua ja tyypitystä

## Hyödyllisiä vinkkejä jatkokehitykseen

### Structured Output Gemini AI:lla

Gemini AI tukee strukturoitua JSON-muotoista tulostetta. Tämä voi olla hyödyllinen tapa saada johdonmukaisia ja helposti käsiteltäviä vastauksia. Voit määritellä JSON-skeeman joko promptissa tai mallin konfiguraatiossa.

Esimerkki skeeman määrittelystä mallin konfiguraatiossa:

```javascript
const schema = {
  type: SchemaType.OBJECT,
  properties: {
    price: {
      type: SchemaType.NUMBER,
      description: "Estimated price of the furniture",
    },
    confidence: {
      type: SchemaType.STRING,
      description: "Confidence level of the estimation",
    },
  },
  required: ["price", "confidence"],
};

const model = genAI.getGenerativeModel({
  model: "gemini-1.5-pro",
  generationConfig: {
    responseMimeType: "application/json",
    responseSchema: schema,
  },
});
```

Lisätietoja: [Gemini API Structured Output Documentation](https://ai.google.dev/gemini-api/docs/structured-output?lang=node)

### Vercel AI SDK

Vercel AI SDK tarjoaa myös hyvän tavan tuottaa strukturoitua dataa. Se käyttää `zod`-kirjastoa skeemojen määrittelyyn ja validointiin.

Esimerkki tekstin analysoinnista:

```javascript
import { generateObject } from "ai";
import { openai } from "@ai-sdk/openai";
import { z } from "zod";

const result = await generateObject({
  model: openai("gpt-4-turbo"),
  schema: z.object({
    price: z.number(),
    confidence: z.string(),
  }),
  prompt: "Estimate the price of this furniture...",
});
```

Esimerkki kuvan analysoinnista:

```javascript
import { generateObject } from "ai";
import { openai } from "@ai-sdk/openai";
import { z } from "zod";
import fs from "fs";
import dotenv from "dotenv";

dotenv.config();

async function analyzeFurnitureImage() {
  const { object } = await generateObject({
    model: openai("gpt-4o-2024-08-06"),
    maxTokens: 512,
    schema: z.object({
      furniture: z.object({
        type: z.string(),
        material: z.string(),
        color: z.string(),
        estimatedPrice: z.number(),
        condition: z.string(),
      }),
    }),
    messages: [
      {
        role: "user",
        content: [
          {
            type: "text",
            text: "Analyze this furniture image and provide details about it.",
          },
          {
            type: "image",
            image: fs.readFileSync("./path/to/your/furniture/image.jpg"),
          },
        ],
      },
    ],
  });

  console.log(JSON.stringify(object, null, 2));
}

analyzeFurnitureImage();
```

Tässä esimerkissä:

1. Käytämme `gpt-4o-2024-08-06` mallia, joka pystyy analysoimaan kuvia.
2. Määrittelemme skeeman `zod`:in avulla, joka kuvaa odotettua vastausta.
3. Lähetämme sekä tekstimuotoisen ohjeen että kuvan analysoitavaksi.
4. Saamme takaisin strukturoidun objektin, joka sisältää tietoja huonekalusta.

Huomaa, että tarvitset OpenAI API avaimen tätä esimerkkiä varten, ja sinun tulee asentaa tarvittavat riippuvuudet (`@vercel/ai`, `openai`, `zod`).

Lisätietoja: [Vercel AI SDK Documentation](https://sdk.vercel.ai/docs)

### Kuvien esikäsittely

Kuvien esikäsittely ennen niiden lähettämistä AI-mallille voi parantaa analyysin tarkkuutta ja nopeutta. Sharp-kirjasto on erinomainen työkalu tähän tarkoitukseen.

Esimerkki Sharp-kirjaston käytöstä:

```javascript
import sharp from "sharp";

const processedImage = await sharp("input.jpg").resize(800, 600).toBuffer();
```

Lisätietoja: [Sharp Documentation](https://sharp.pixelplumbing.com/)

Nämä työkalut ja tekniikat voivat merkittävästi parantaa sovelluksen toiminnallisuutta ja tehokkuutta jatkokehityksessä.

## LLM-mallien käyttö ja promptien optimointi

Tässä projektissa käytetään LLM-malleja (Large Language Models) kuten Gemini ja OpenAI:n GPT-malleja. Tässä on muutamia hyödyllisiä vinkkejä näiden mallien tehokkaaseen käyttöön:

### 1. Promptien suunnittelu

- **Ole tarkka ja yksityiskohtainen**: Mitä tarkempi olet promptissasi, sitä parempia tuloksia saat.
- **Kontekstin antaminen**: Anna mallille riittävästi kontekstia tehtävästä ja halutusta lopputuloksesta.
- **Esimerkkien käyttö**: Few-shot learning -tekniikka, jossa annat mallille esimerkkejä halutusta vastauksesta, voi parantaa tuloksia huomattavasti.

Esimerkki parannellusta promptista:

```javascript
const prompt = `
Olet asiantuntija huonekalujen arvioinnissa. Analysoi seuraava huonekalu:
Tyyppi: ${furniture.type}
Merkki: ${furniture.brand}
Malli: ${furniture.model}
Materiaali: ${furniture.material}
Kunto: ${furniture.condition}
Ikä: ${furniture.age} vuotta

Anna hinta-arvio euroina ja perustele se lyhyesti. Ota huomioon Suomen markkinatilanne.

Esimerkki vastauksesta:
Hinta-arvio: 450 €
Perustelu: Kyseessä on hyväkuntoinen, tunnetun valmistajan tuote. Ikä ja pienet käytön jäljet laskevat hieman arvoa, mutta klassinen design pitää hinnan kohtuullisena.

Anna nyt vastauksesi tästä huonekalusta:
`;
```

### 2. Mallin valinta ja parametrit

- **Mallin valinta**: Geminin ja OpenAI:n eri malleilla on erilaisia vahvuuksia. Kokeile eri malleja löytääksesi parhaiten tehtävään sopivan.
- **Lämpötila (Temperature)**: Matalampi lämpötila (lähellä 0) tuottaa johdonmukaisempia vastauksia, korkeampi (lähellä 1) luovempia.
- **Maksimitokenit**: Säädä tätä tarpeen mukaan, mutta muista, että pidempi vastaus ei aina ole parempi.

Esimerkki mallin konfiguroinnista Geminillä:

```javascript
const model = genAI.getGenerativeModel({
  model: "gemini-1.5-pro",
  generationConfig: {
    temperature: 0.3,
    maxOutputTokens: 500,
  },
});
```

### 3. Vastausten käsittely ja validointi

- **Strukturoitu data**: Pyydä mallia tuottamaan vastaukset strukturoidussa muodossa (esim. JSON) helpompaa jatkokäsittelyä varten.
- **Validointi**: 
  - Strukturoidun datan kanssa: Jos käytät ennalta määriteltyä skeemaa tai työkalua, joka pakottaa mallin tuottamaan strukturoidun tuloksen, voit vähentää tai jopa poistaa erillisen validoinnin tarpeen.
  - Vapaamuotoisen datan kanssa: On hyvä tarkistaa mallin tuottamat vastaukset. Käytä esimerkiksi Zod-kirjastoa JSON-vastausten validointiin.

### 4. Jatkuva parantaminen

- **Vastausten arviointi**: Seuraa mallien suorituskykyä ja kerää palautetta käyttäjiltä.
- **Promptien iterointi**: Hienosäädä prompteja saadun palautteen perusteella jatkuvasti.
- **RAG (Retrieval-Augmented Generation)**: Harkitse RAG-mallin käyttöönottoa oman datan integroimiseksi LLM:n vastauksiin. Tämä on usein tehokkaampi ja joustavampi vaihtoehto kuin perinteinen fine-tuning.

#### RAG-mallin käyttö

RAG yhdistää tiedonhaun (retrieval) ja tekstin generoinnin, mahdollistaen LLM:n käytön yhdessä ulkoisen tietokannan kanssa. Tämä on erityisen hyödyllistä, kun haluat yhdistää yleisen LLM:n tietämyksen omaan erikoistuneeseen dataasi.

RAG:in etuja:

- **Ajantasaisuus**: Voit päivittää tietokantaa ilman mallin uudelleenkoulutusta.
- **Läpinäkyvyys**: Voit jäljittää, mistä tieto on peräisin.
- **Kustannustehokkuus**: Usein edullisempi kuin koko mallin fine-tuning.

Esimerkki RAG:in implementoinnista:

```javascript
import { OpenAI } from 'langchain/llms/openai';
import { VectorStore } from 'langchain/vectorstores/base';
import { OpenAIEmbeddings } from 'langchain/embeddings/openai';

async function ragQuery(query, vectorStore) {
  // 1. Hae relevantit dokumentit vektoritietokannasta
  const relevantDocs = await vectorStore.similaritySearch(query, 3);

  // 2. Muodosta konteksti relevanteista dokumenteista
  const context = relevantDocs.map(doc => doc.pageContent).join('\n');

  // 3. Luo prompti, joka yhdistää kyselyn ja kontekstin
  const prompt = `
  Konteksti: ${context}

  Kysymys: ${query}

  Anna vastaus perustuen annettuun kontekstiin ja yleiseen tietämykseesi.
  `;

  // 4. Lähetä prompti LLM:lle
  const llm = new OpenAI();
  const response = await llm.predict(prompt);

  return response;
}

// Käyttöesimerkki
const vectorStore = // ... alusta vektoritietokanta omalla datallasi
const query = "Mikä on tyypillinen hinta Artekin jakkara 60:lle?";
const answer = await ragQuery(query, vectorStore);
console.log(answer);
```

Tämä lähestymistapa mahdollistaa LLM:n vastausten rikastamisen omalla datasetillasi, mikä on erityisen hyödyllistä huonekalujen hinnoittelussa ja arvioinnissa, missä markkinatilanne ja hinnat voivat vaihdella alueittain ja ajan myötä.

Nämä vinkit auttavat sinua hyödyntämään LLM-malleja tehokkaammin projektissasi ja tuottamaan tarkempia ja luotettavampia tuloksia huonekalujen arvioinnissa.

## API-hinnoittelu: Yksinkertaisia esimerkkejä (Syyskuu 2024)

Nämä esimerkit perustuvat GPT-4o-2024-08-06 -mallin hinnoitteluun syyskuussa 2024. Käytämme normaalia hinnoittelua, ei Batch API -hintoja.

### Tekstipohjainen kysely

Esimerkki: Huonekalun hinta-arvio tekstikuvauksen perusteella

```
Kysely: "Anna hinta-arvio ruskealle nahkasohvalle, joka on 3 vuotta vanha ja hyvässä kunnossa."
Vastaus: "Perustuen annettuihin tietoihin, arvioisin ruskean nahkasohvan hinnaksi noin 600-800 euroa käytettynä..."
```

Arvioitu kustannus:

- Syöte: ~30 tokenia ≈ $0.000075 (€0.000069)
- Tuloste: ~50 tokenia ≈ $0.0005 (€0.00046)
- Yhteensä: $0.000575 (€0.00053) (noin 0,05 senttiä)

### Kuvapohjainen kysely

Esimerkki: Huonekalun tunnistus ja hinta-arvio kuvan perusteella

```
[Kuva ruskeasta nahkasohvasta]
Kysely: "Mikä huonekalu tämä on ja paljonko se voisi maksaa?"
Vastaus: "Kuva esittää ruskeaa nahkasohvaa. Se näyttää olevan hyvässä kunnossa..."
```

Arvioitu kustannus:

- Kuva (oletetaan 1024x768px): $0.000638 (€0.00059)
- Tekstisyöte: ~10 tokenia ≈ $0.000025 (€0.000023)
- Tuloste: ~50 tokenia ≈ $0.0005 (€0.00046)
- Yhteensä: $0.001163 (€0.00107) (noin 0,1 senttiä)

### Kokonaiskustannusarvio tyypilliselle käyttötapaukselle

Oletetaan, että sovelluksesi analysoi 100 huonekalua päivässä, joista puolet tekstipohjaisesti ja puolet kuvapohjaisesti:

- 50 tekstipohjaista kyselyä: 50 \* €0.00053 = €0.0265
- 50 kuvapohjaista kyselyä: 50 \* €0.00107 = €0.0535
- Päivittäinen kokonaiskustannus: €0.08 (noin 8 senttiä)

Kuukausikustannus (30 päivää): €2.40

### Huomioitavaa

1. Nämä hinnat perustuvat GPT-4o-2024-08-06 -malliin, joka on edullisempi kuin aiemmat versiot.
2. Todelliset kustannukset voivat vaihdella riippuen tarkan kyselyn ja vastauksen pituudesta sekä kuvan koosta.
3. Suuremmilla volyymeilla voi olla mahdollista käyttää Batch API -hinnoittelua, joka puolittaisi kustannukset.
4. Hinnat voivat muuttua ajan myötä, joten tarkista aina viimeisimmät hinnat OpenAI:n sivustolta.

Nämä esimerkit osoittavat, että GPT-4o-mallin käyttö on suhteellisen edullista. Yksittäiset kyselyt maksavat vain murto-osia sentistä, mikä mahdollistaa laajan käytön ilman merkittäviä kustannuksia.

## Ramdom tietoa | Hyödyllisiä työkaluja RAG-toteutukseen

### LangChain

LangChain on monipuolinen sovelluskehys LLM-pohjaisten sovellusten rakentamiseen. Se on saatavilla sekä Python- että Node.js-ympäristöihin, mikä tekee siitä joustavan vaihtoehdon eri kehitysympäristöihin. LangChain tarjoaa useita hyödyllisiä ominaisuuksia:

- Promptien hallinta ja ketjutus
- Muistikomponentit keskusteluhistorian hallintaan
- Integraatiot eri vektoritietokantoihin ja hakumoottoreihin
- Valmiit työkalut RAG-toteutuksiin

LangChainin avulla voit helposti yhdistää eri LLM-malleja, tietokantoja ja muita komponentteja kokonaisvaltaisiksi AI-sovelluksiksi.

### Supabase

Supabase on avoimen lähdekoodin Backend-as-a-Service (BaaS) alusta, joka tarjoaa myös vektoritietokantaominaisuuksia. Se on rakennettu PostgreSQL:n päälle ja tarjoaa ilmaisen tason kehittäjille. Supabasen vektoritietokanta-ominaisuudet tekevät siitä houkuttelevan vaihtoehdon RAG-toteutuksiin:

- Helppo integroida olemassa oleviin sovelluksiin
- Tuki pgvector-laajennukselle, mikä mahdollistaa tehokkaan vektorihakujen
- Skaalautuva ratkaisu, joka kasvaa projektisi mukana
- Yhdistää perinteisen relaatiotietokannan ja vektoritietokannan edut

Supabase tarjoaa myös muita hyödyllisiä palveluita, kuten autentikaation ja reaaliaikaisen tiedonsiirron, mikä tekee siitä monipuolisen alustan kokonaisvaltaisten sovellusten rakentamiseen.

### Datan kerääminen

RAG-mallien tehokas käyttö edellyttää usein oman erikoistuneen datan keräämistä. Web scraping on yksi yleinen menetelmä tähän tarkoitukseen. Tässä muutamia työkaluja ja huomioita web scrapingiin liittyen:

- **Python-työkalut**: 
  - BeautifulSoup: Helppokäyttöinen kirjasto HTML- ja XML-tiedostojen jäsentämiseen
  - Scrapy: Tehokas ja skaalautuva web crawling -sovelluskehys

- **Node.js-työkalut**:
  - Cheerio: Nopea ja joustava kirjasto HTML:n manipulointiin
  - Puppeteer: Googlen kehittämä työkalu, joka mahdollistaa selainpohjaisen scrapingin

Web scraping on tehokas tapa kerätä dataa, mutta huomioi, että jotkin sivustot saattavat rajoittaa tai kieltää scraping-toiminnan.
