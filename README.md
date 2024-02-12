--SELECT *
--FROM Projects.dbo. COVIDDeathRates
--Order by 3,4

--SELECT 
--FROM Projects.dbo. COVIDVaccinations
--Order by 3,4

-- View data from the COVIDVaccination table
--SELECT *
--FROM Projects.dbo.COVIDVaccinations;

-- View data from the COVIDDeathRates table
--SELECT *
--FROM Projects.dbo.COVIDDeathRates;

--Select Data that we are going to be using 

--SELECT Location, date, total_cases, new_cases, total_deaths, population
--FROM Projects.dbo.[COVIDDeathRates]
--ORDER BY 1, 2;

--Looking at the Total Cases VS Total Deaths
--Shows  likehood of dying if contracted covid in your country
--SELECT Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage 
--FROM Projects.dbo.COVIDDeathRates
--Where location like '%Africa%'
--ORDER BY 1, 2;

--looking at total casese VS populcation
--Shows what percetnage of population got COVID
--SELECT Location, date, total_cases, Population, (total_cases/population)*100 as PercentagePopulationInfected
--FROM Projects.dbo.COVIDDeathRates
--Where location like '%Africa%'
--ORDER BY 1, 2;

--Looking at Countries with the highest infection rate compared to population 
--SELECT Location, Population,  MAX(total_cases)as  HighesInfectionCOunt, MAX(Total_cases/population)*100 as PercentagePopulationInfected
--FROM Projects.dbo.COVIDDeathRates
----Where location like '%Africa%'
--Group by location , Population
--ORDER BY PercentagePopulationInfected desc

--Show Countries with the highest Death count per Population
--SELECT Location, MAX(cast(Total_deaths as int)) as TotalDeathCOunt
--FROM Projects.dbo.COVIDDeathRates
----Where location like '%Africa%'
--Group by Location
--ORDER BY TotalDeathCOunt desc


--Let break things down by Continent
--SELECT Continent, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
--FROM Projects.dbo.COVIDDeathRates
--GROUP BY Continent
--ORDER BY TotalDeathCount DESC;

-- Global Numbres 
--SELECT 
--    SUM(new_deaths) as total_deaths,
--    SUM(cast(new_deaths as int)) as total_deaths,
--    (SUM(cast(new_deaths as int)) / NULLIF(SUM(new_deaths), 0)) * 100 as DeathPercentage
--FROM 
--    Projects.dbo.COVIDDeathRates
--WHERE 
--    continent IS NOT NULL
--ORDER BY 
--    1, 2;

-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

--Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
--, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
----, (RollingPeopleVaccinated/population)*100
--From Projects. dbo.COVIDDeathRates dea
--Join Projects.dbo.COVIDVaccinations vac
--	On dea.location = vac.location
--	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

-- Using CTE to perform Calculation on Partition By in previous query
--WITH PopvsVac (Continent, Location, Date, Population, new_vaccinations, RollingPeopleVaccinated)
--AS
--(
--    SELECT 
--        dea.continent, 
--        dea.location, 
--        dea.date, 
--        dea.population, 
--        vac.new_vaccinations,
--        SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
--    FROM 
--        Projects.dbo.COVIDDeathRates dea
--    JOIN 
--        Projects.dbo.COVIDVaccinations vac ON dea.location = vac.location
--                                             AND dea.date = vac.date
--    WHERE 
--        dea.continent IS NOT NULL 
--)
--SELECT 
--    *,
--    (RollingPeopleVaccinated / Population) * 100 AS VaccinationPercentage
--FROM 
--    PopvsVac;

-- Using Temp Table to perform Calculation on Partition By in previous query

--DROP Table if exists #PercentPopulationVaccinated
--Create Table #PercentPopulationVaccinated
--(
--Continent nvarchar(255),
--Location nvarchar(255),
--Date datetime,
--Population numeric,
--new_vaccinations numeric,
--RollingPeopleVaccinated numeric
--)

--Insert into #PercentPopulationVaccinated
--Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
--, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
----, (RollingPeopleVaccinated/population)*100
--From Projects.dbo.COVIDDeathRates dea
--Join Projects.dbo.COVIDVaccinations vac
--	On dea.location = vac.location
--	and dea.date = vac.date
----where dea.continent is not null 
----order by 2,3

--Select *, (RollingPeopleVaccinated/Population)*100
--From #PercentPopulationVaccinated


-- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From Projects.dbo.COVIDDeathRates dea
Join Projects.dbo.COVIDVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
