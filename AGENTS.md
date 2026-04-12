# Retningslinjer
- Når e sesjon starter så sjekk git status om alle filer og endringer er committed i git.
- Alle script skal ha et --silence argument, som da undertrykker skriving til stdout. Kun feilmeldinger skal skrives ut.
- Alle py script skal lages med --dryrun opsjon. Da skrives ingenting til database eller fil, men det gjør nødvendig
les.
- Når et py script er laget eller endret skal det test kjøres, med --dryrun opsjon og sjekk output.
- Ved endringer i tabell skjema, så lag en egen fil på det i database/migration/ katalogen. Filen skal hete migration-YYYY-MM-DD.sql hvor YYYY-MM-DD er dagens dato. I tillegg må tabell definisjonen i database katalogen.


# /export
Legges opencode katalogen i repoet.

.