# Covid Data
In this project, we are conducting Covid-19 Data Exploration.
Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types

```
SELECT *
FROM 
	PortfolioProject..CovidDeaths
WHERE 
	continent is NOT NULL 
ORDER BY 
	location, date
```

#### Selecting data that we are going to be starting with:
```
SELECT 
	location, date, total_cases, new_cases, total_deaths, population
FROM 
	PortfolioProject..CovidDeaths
WHERE
	continent is NOT NULL
ORDER BY 
	1,2
```

-- Total Cases vs Total Deaths
#### Likelihood of dying if you contract covid in your country:
```
SELECT 
	location, date, total_cases,total_deaths
	, (total_deaths/total_cases)*100 AS DeathPercentage
FROM
	PortfolioProject..CovidDeaths
WHERE 
	location LIKE '%states%'
	AND continent is not null 
ORDER BY 
	1,2
```

-- Total Cases vs Population
#### Shows what percentage of population infected with Covid:

```
SELECT 
	location, date, Population, total_cases,
	(total_cases/population)*100 AS PercentPopulationInfected
FROM
	PortfolioProject..CovidDeaths
--Where location like '%states%'
ORDER BY 1,2
```

#### Countries with Highest Infection Rate compared to Population
```
SELECT
	Location, Population, MAX(total_cases) AS HighestInfectionCount,  MAX((total_cases/population))*100 as PercentPopulationInfected
FROM
	PortfolioProject..CovidDeaths
--Where location like '%states%'
GROUP BY
	location, population
ORDER BY
	PercentPopulationInfected DESC
```

#### Countries with Highest Death Count per Population:
```
SELECT
	Location, MAX(cast(Total_deaths as int)) AS TotalDeathCount
FROM
	PortfolioProject..CovidDeaths
--Where location like '%states%'
WHERE
	continent is NOT NULL
GROUP BY
	Location
ORDER BY
	TotalDeathCount DESC
```


-- BREAKING THINGS DOWN BY CONTINENT

#### Showing contintents with the highest death count per population:

```
SELECT
	continent, 
	MAX(cast(Total_deaths as int)) AS TotalDeathCount
FROM 
	PortfolioProject..CovidDeaths
--Where location like '%states%'
WHERE
	continent is NOT NULL 
GROUP BY continent
ORDER BY TotalDeathCount DESC
```


#### GLOBAL NUMBERS
```
SELECT
	SUM(new_cases) as total_cases, 
	SUM(cast(new_deaths as int)) as total_deaths, 
	SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
FROM 
	PortfolioProject..CovidDeaths
--Where location like '%states%'
WHERE 
	continent is NOT NULL 
--Group By date
ORDER BY
	1,2
```


## Total Population vs Vaccinations
#### Shows Percentage of Population that has recieved at least one Covid Vaccine:
```
SELECT 
	dea.continent, dea.location, dea.date, dea.population, 
	vac.new_vaccinations, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
FROM 
	PortfolioProject..CovidDeaths dea
JOIN 
	PortfolioProject..CovidVaccinations vax
		ON dea.location = vac.location
		AND dea.date = vac.date
WHERE
	dea.continent is NOT NULL 
ORDER BY
	2,3
```

#### Using CTE to perform Calculation on PARTITION BY in previous query:
```
WITH
	POPvsVAX
		(Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
AS
(
	select 
		dea.continent, dea.location, dea.date, dea.population, 
		vac.new_vaccinations, SUM(CONVERT(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
	from
		PortfolioProject..CovidDeaths dea
	join 
		PortfolioProject..CovidVaccinations vac
			on dea.location = vac.location
			and dea.date = vac.date
	where 
		dea.continent is not null 
--order by 2,3
)
SELECT *,
	(RollingPeopleVaccinated/Population)*100
FROM 
	POPvsVAX
```


#### Using Temp Table to perform Calculation on Partition By in previous query
```
DROP TABLE 
	IF EXISTS 
    #VaccinatedPercentile
  CREATE TABLE
	  #VaccinatedPercentile
(
	Continent nvarchar(255),
	Location nvarchar(255),
	Date datetime,
	Population numeric,
	New_vaccinations numeric,
	RollingPeopleVaccinated numeric
)

INSERT
	into #VaccinatedPercentile
SELECT
	dea.continent, dea.location, dea.date, dea.population, 
	vac.new_vaccinations, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
FROM 
	PortfolioProject..CovidDeaths dea
JOIN 
	PortfolioProject..CovidVaccinations vac
		ON dea.location = vac.location
		AND dea.date = vac.date
--where dea.continent is not null 
--order by 2,3
```

```
SELECT *, 
	(RollingPeopleVaccinated/Population)*100
FROM 
	#VaccinatedPercentile
```


#### Creating View to store data for later visualizations:

```
CREATE VIEW
	PercentPopulationVaccinated as
SELECT 
	dea.continent, dea.location, dea.date, dea.population, 
	vac.new_vaccinations, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
FROM 
	PortfolioProject..CovidDeaths dea
JOIN 
	PortfolioProject..CovidVaccinations vac
		ON dea.location = vac.location
		AND dea.date = vac.date
WHERE
	dea.continent is NOT NULL
  ```
