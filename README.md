# Multiplication Trainer Infrastructure

Deze repository bevat de Terraform infrastructuur voor de [Multiplication Trainer applicatie](https://github.com/edwinbulter/multiplication-trainer). De infrastructuur zorgt voor het hosten van de statische website op AWS met behulp van S3 en CloudFront.

**🌐 Live Demo**: Een werkende versie van de applicatie is beschikbaar op [https://d2zf8l8tihew58.cloudfront.net/](https://d2zf8l8tihew58.cloudfront.net/)

## Inhoudsopgave

- [Installatie Instructies](#installatie-instructies)
  - [Vereisten](#vereisten)
  - [Terraform](#terraform)
  - [AWS CLI](#aws-cli)
- [Overzicht van de Infrastructuur](#overzicht-van-de-infrastructuur)
  - [S3 Bucket](#s3-bucket)
  - [CloudFront Distribution](#cloudfront-distribution)
- [Gebruik](#gebruik)
- [Variabelen](#variabelen)
- [Outputs](#outputs)
- [Onderhoud](#onderhoud)
- [Licentie](#licentie)

## Installatie Instructies

### Vereisten

Om met deze infrastructuur te werken, heb je de volgende zaken nodig:

#### Terraform
Infrastructuur als code tool voor het beheren van cloud resources

```bash
# Installeer Terraform met Homebrew
brew install terraform

# Controleer de installatie
terraform --version
```

#### AWS CLI
Command-line interface voor interactie met AWS diensten

```bash
# Installeer AWS CLI met Homebrew
brew install awscli

# Controleer de installatie
aws --version

# Configureer AWS CLI (volg de instructies)
aws configure
```
**Let op**: Je hebt hiervoor je AWS Access Key ID en Secret Access Key nodig.

## Overzicht van de Infrastructuur

Deze Terraform configuratie creëert de volgende AWS resources:

### S3 Bucket
- **Doel**: Opslag van de statische website bestanden (HTML, CSS, JavaScript)
- **Configuratie**: Website hosting ingeschakeld met `index.html` als hoofdpagina
- **Bucket naam**: Configureerbaar via variabele (standaard: `multiplication-trainer-bucket`)

### CloudFront Distribution
- **Doel**: Content Delivery Network (CDN) voor snelle wereldwijde toegang
- **Features**:
  - HTTPS redirect (alle HTTP verkeer wordt doorgestuurd naar HTTPS)
  - IPv6 ondersteuning
  - Caching van statische bestanden
  - Geografische restricties: geen (wereldwijd toegankelijk)

### Origin Access Identity (OAI)
- **Doel**: Beveiligde toegang van CloudFront naar de S3 bucket
- **Functie**: Voorkomt directe toegang tot S3 bucket, verkeer moet via CloudFront

### S3 Bucket Policy
- **Doel**: Geeft CloudFront toestemming om bestanden uit de S3 bucket te lezen
- **Beveiliging**: Alleen CloudFront kan de bucket-inhoud benaderen

## Variabelen

| Variabele | Beschrijving | Standaardwaarde |
|-----------|--------------|-----------------|
| `aws_region` | AWS regio waar resources worden aangemaakt | `eu-central-1` |
| `bucket_name` | Naam van de S3 bucket | `multiplication-trainer-bucket` |

## Outputs

Na succesvolle deployment krijg je de volgende outputs:
- `s3_bucket_name`: De naam van de aangemaakte S3 bucket
- `cloudfront_url`: De CloudFront distributie URL waar de website bereikbaar is

## Installatie Instructies

### Stap-voor-stap installatie

1. **Clone deze repository**
   ```bash
   git clone <repository-url>
   cd multiplication-trainer-infrastructure
   ```

2. **Controleer AWS configuratie**
   ```bash
   aws configure list --profile edwinbulter
   ```

3. **Initialiseer Terraform**
   ```bash
   terraform init
   ```

4. **Controleer het deployment plan**
   ```bash
   terraform plan
   ```

5. **Deploy de infrastructuur**
   ```bash
   terraform apply
   ```
   Bevestig met `yes` wanneer gevraagd.

6. **Noteer de outputs**
   Na succesvolle deployment zie je:
   - S3 bucket naam
   - CloudFront URL

### Website bestanden uploaden

Na het aanmaken van de infrastructuur moet je de website bestanden uploaden naar de S3 bucket:

```bash
# Upload bestanden naar S3 bucket
aws s3 sync /pad/naar/website/bestanden s3://multiplication-trainer-bucket --profile edwinbulter
```

### Infrastructuur verwijderen

Om alle resources te verwijderen:
```bash
terraform destroy
```

## Belangrijke opmerkingen

- **Kosten**: CloudFront en S3 hebben variabele kosten gebaseerd op gebruik
- **DNS**: De CloudFront URL is direct bruikbaar, maar je kunt ook een custom domain configureren
- **Caching**: CloudFront cached bestanden, bij updates kan het even duren voordat wijzigingen zichtbaar zijn
- **Beveiliging**: De S3 bucket is alleen toegankelijk via CloudFront, niet direct

## Gerelateerde Repository

De broncode van de Multiplication Trainer applicatie vind je hier: [https://github.com/edwinbulter/multiplication-trainer](https://github.com/edwinbulter/multiplication-trainer)
