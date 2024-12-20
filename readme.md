# GPU Module Repository

## Description
Ce repository contient le module `GPU` conçu pour gérer les connexions et récupérer des informations depuis le site GPU. Il inclut des fonctions pour se connecter, obtenir des informations sur les étudiants, lister les semaines disponibles et télécharger les données d'une semaine spécifique. Il contient également des classes pour représenter des événements et des étudiants, ainsi qu'un agenda pour gérer ces événements.

## Installation
Pour utiliser ce module, assurez-vous que tous les fichiers nécessaires sont dans le même répertoire et que vous avez configuré les variables d'environnement si nécessaire.

### Pré-requis
- Python 3.x
- Modules `re`,`requests`, `BeautifulSoup`, et `datetime`

## Contenu du Module

### Fichiers et Fonctions Principales

- **fonctions.py**
  - `get_connection(login: int, password: str='123') -> requests.Session`: Connexion à GPU via une requête POST et retourne la session avec les cookies.
  - `get_infos(session: requests.Session) -> tuple[str, int, int]`: Récupère les informations de l'étudiant en utilisant la session active.
  - `list_semaine(session: requests.Session) -> list[int]`: Récupère la liste des semaines disponibles à partir de la page d'agenda étudiant.
  - `get_Semaine(num_etu: int, week: int, session: requests.Session, file_path: str) -> None`: Télécharge les données d'une semaine donnée et les enregistre dans un fichier VCS.
  - `get_Semaine_Pdf(num_etu: int, week: int, year: int, session: requests.Session, file_path: str) -> None`:Télécharge les données d'une semaine donnée et les enregistre dans un fichier PDF.


- **class_perso.py**
  - `Event`: Classe représentant un événement iCalendar (RFC 5545).
    - `__init__(self, sequence: int, uid: str, dtstamp: datetime.datetime, description: str, summary: str, location: str, dtstart: datetime.datetime, dtend: datetime.datetime, priority: int, event_class: str) -> None`: Initialise un événement avec tous ses attributs.
    - `__str__(self) -> str`: Retourne une chaîne lisible représentant l'événement.
    - `format_datas(self) -> None`: Restructure les données de l'événement.
    - `ajouter_heure_si_hiver(self) -> None`: Rajoute une heure au début et à la fin d'un événement si celui-ci est dans l'heure d'hiver.
    - `parse_datetime(ical_date: str) -> datetime.datetime`: Convertit une chaîne de date au format iCalendar en objet datetime.
    - `from_ical(cls, ical_data: dict[str, str]) -> 'Event'`: Crée une instance d'Event à partir des données iCalendar.
  - `Student`: Classe représentant un étudiant.
    - `__init__(self, num: int, nom: Optional[str] = None, td: Optional[int] = None, tp: Optional[int] = None) -> None`: Initialise un nouvel étudiant avec un numéro, et éventuellement un nom, un groupe de TD et un groupe de TP.
    - `__repr__(self) -> str`: Retourne une représentation de l'étudiant.
    - `_completer_infos(self) -> None`: Complète les informations de l'étudiant en utilisant les fonctions get_connection et get_infos.
  - `Agenda`: Classe représentant un agenda pour un étudiant.
    - `__init__(self, student: Student) -> None`: Initialise un nouvel agenda pour un étudiant.
    - `__len__(self) -> int`: Renvoie le nombre d'événements dans l'agenda.
    - `ajouter_event(self, event: Event) -> None`: Ajoute un événement à l'agenda.
    - `ajouter_events(self, events: list[Event]) -> None`: Ajoute plusieurs événements à l'agenda.
    - `__repr__(self) -> str`: Retourne une représentation sous forme de chaîne de l'agenda.
    - `restructuration(self) -> None`: Restructure les données des événements.
    - `export(self, filename: str) -> None`: Exporte l'agenda dans un fichier iCalendar.

## Utilisation

### Exemple d'Utilisation

```python
import gpulr as GPU
import os,sys

def main(etudiant: GPU.Student) -> None:
    # Définition des paramètres
    num_etu = etudiant.num
    etu_directory = str(num_etu)
    temp_dir_week = f"{etu_directory}/week_temp"

    print("Démarrage du script")
    # Création des répertoires
    os.makedirs(etu_directory, exist_ok=True)
    os.makedirs(temp_dir_week, exist_ok=True)

    try:
        print('Initialisation de la connexion à GPU')
        user_session = GPU.get_connection(num_etu)
    except ConnectionError:
        print('ConnectionError: Échec de la connexion')
        return

    print('Connecté avec succès')

    semaines_dispo = GPU.list_semaine(user_session)
    print(f'Liste des semaines disponibles : {semaines_dispo}')

    for numero in semaines_dispo:
        print(f'Téléchargement de la semaine {numero}')
        GPU.get_Semaine(num_etu,numero,user_session,f"{temp_dir_week}/week_{numero}")

    agenda = GPU.Agenda(etudiant)
    for file in os.listdir(temp_dir_week):
        file_path = os.path.join(temp_dir_week, file)
        with open(file_path, 'r') as f:
            agenda.ajouter_events(GPU.parse_multiple_events(f.readlines()))
    
    
    print("Restructuration des événements")
    agenda.restructuration()

    print(f"Export de l'agenda dans le fichier {etu_directory}/{num_etu}.ics")
    agenda.export(f"{etu_directory}/{num_etu}.ics")

if __name__ == "__main__":
    # Passer le numéro étudiant comme argument
    if len(sys.argv) != 2:
        print("Usage: python script.py <num_etu>")
        sys.exit(1)
    main(GPU.Student(int(sys.argv[1])))
```

