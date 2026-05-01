# COVID-19 Data Exploration using SQL

## Project Overview

This project performs end-to-end exploratory data analysis (EDA) on global COVID-19 data using SQL Server. The analysis covers infection rates, death percentages, vaccination rollouts, and country/continent-level comparisons — transforming raw public health data into meaningful insights.

> This is a portfolio project focused on demonstrating core SQL skills in a real-world dataset context.

---

## Dataset

- **Source:** [Our World in Data – COVID-19 Deaths](https://ourworldindata.org/covid-deaths)
- **Underlying data provider:** World Health Organization (WHO)
- **Download link:** [GitHub – owid/covid-19-data](https://github.com/owid/covid-19-data/tree/master/public/data)
- **Format:** CSV (split into CovidDeaths and CovidVaccinations tables for this project)
- **Authors:** Edouard Mathieu, Hannah Ritchie, Lucas Rodés-Guirao et al. — Our World in Data

---

## SQL Skills Demonstrated

| Skill | Usage |
|---|---|
| JOINs | Combining Deaths and Vaccinations tables |
| CTEs | Calculating rolling vaccination percentages |
| Temp Tables | Storing intermediate results for reuse |
| Window Functions | Running totals with OVER (PARTITION BY ...) |
| Aggregate Functions | SUM, MAX for country/continent summaries |
| Views | Creating reusable views for visualization tools |
| Data Type Conversion | CAST, CONVERT for numeric operations |

---

## Key Analysis Performed

### 1. Death Percentage by Country
*"If you contracted COVID in the US, what was the likelihood of dying?"*

```sql
SELECT Location, date, total_cases, total_deaths,
       (total_deaths / total_cases) * 100 AS DeathPercentage
FROM CovidDeaths
WHERE location LIKE '%states%'
```

### 2. Population Infection Rate
*"What percentage of each country's population was infected?"*

```sql
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount,
       MAX((total_cases / population)) * 100 AS PercentPopulationInfected
FROM CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC
```

### 3. Countries with Highest Death Count

```sql
SELECT Location, MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC
```

### 4. Continent-Level Death Analysis

```sql
SELECT continent, MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC
```

### 5. Rolling Vaccination Progress (CTE)
*"What percentage of the population has received at least one vaccine dose over time?"*

```sql
WITH PopvsVac AS (
  SELECT dea.location, dea.date, dea.population, vac.new_vaccinations,
         SUM(CONVERT(INT, vac.new_vaccinations))
           OVER (PARTITION BY dea.Location ORDER BY dea.date) AS RollingPeopleVaccinated
  FROM CovidDeaths dea
  JOIN CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
  WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated / Population) * 100
FROM PopvsVac
```

---

## Repository Structure

```
covid-sql-exploration/
│
├── covid_data_exploration.sql   -- Main SQL script with all queries
└── README.md                    -- Project documentation
```

---

## How to Run

1. Download `owid-covid-data.csv` from [Our World in Data GitHub](https://github.com/owid/covid-19-data/tree/master/public/data)
2. Split into two tables — `CovidDeaths` and `CovidVaccinations` — using Excel or Python
3. Import both into SQL Server Management Studio (SSMS)
4. Create a database named `SQLDataExploration`
5. Open `covid_data_exploration.sql` and run queries section by section

---

## Insights Uncovered

- Countries with the highest infection rates relative to population (small European nations showed rates above 50%)
- The global death percentage hovered around 1–2% for most of the pandemic
- Vaccination rollouts were uneven — developed nations vaccinated significantly faster
- Continent-level analysis shows North America and Europe had the highest absolute death counts

---

## Future Scope

- Connect results to Tableau or Power BI for interactive dashboards
- Add time-series analysis to track wave patterns
- Extend with Python (Pandas / Matplotlib) for visual EDA

---

## Author

**Bhavik**
B.E. Information Technology | Walchand Institute of Technology, Solapur
[your-email@example.com] | [LinkedIn Profile] | [GitHub Profile]

---

## License

This project is open-source and available under the [MIT License](LICENSE).

---

*Dataset credit: Edouard Mathieu, Hannah Ritchie, Lucas Rodés-Guirao et al. — Our World in Data. Data sourced from the World Health Organization (WHO).*
