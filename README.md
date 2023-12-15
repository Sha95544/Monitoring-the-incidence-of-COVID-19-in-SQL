# Monitoring the incidence of COVID-19 in SQL

```sql

--Select *
--From PortfolioProject..CovidDeaths$
--Select *
--From PortfolioProject..CovidDeaths
--Where continent is not null
--order by 3,4

--Select *
--From PortfolioProject..CovidVaccinations
--order by 3,4

--Selecting the data that will be used
Select Location, date, total_cases, new_cases, total_deaths, population
From PortfolioProject..CovidDeaths
Where continent is not null
order by 1,2

-- Looking at the % of deaths by country per covid cases:
-- Looking at the likelihood of dying of COVID in Pakistan
-- (total_deaths/total_cases)*100 as Death_Percentage : creates new clumn displaying the %age
 Select Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Death_Percentage
From PortfolioProject..CovidDeaths
Where location = 'Pakistan' and continent is not null
order by 1,2

--Looking at the %age of total COVUD cases per population in Pakistan:
 Select Location, date, total_cases, Population, (total_cases/population)*100 as Percent_Covid_Cases
From PortfolioProject..CovidDeaths
Where location = 'Pakistan' and continent is not null
order by 1,2



--Finding countires with highest instance of COVID per populaation:
--Using GROUP BY to generate summary rows per country from similar rows i.e COVID instance in each country

 Select Location, MAX(total_cases) as Max_COVID_Cases, Population, MAX((total_cases/population))*100 as Percent_Covid_Cases
From PortfolioProject..CovidDeaths
GROUP BY Location, Population
--Where location = 'Pakistan'
order by Percent_Covid_Cases desc


-- Showing Countries with highest death counts per population:
-- using cast as values in total_deaths column show nvarchar.
 Select Location, MAX(cast(total_deaths as int)) as Tot_Death_Count, Population, MAX((total_deaths/population))*100 as Percent_Deaths_per_population
From PortfolioProject..CovidDeaths
WHERE continent is not null
GROUP BY Location, Population
order by Tot_Death_Count desc

--Doing the same for continents
 Select continent, MAX(cast(total_deaths as int)) as Tot_Death_Count
From PortfolioProject..CovidDeaths
WHERE continent is not null
GROUP BY continent
order by Tot_Death_Count desc


--Finding the global stats on each date:
Select date, SUM(new_cases) as Total_Cases, SUM(cast(new_deaths as int)) as Total_Deaths, SUM(cast(new_deaths as int))/SUM(new_cases)*100 as Percent_Deaths
From PortfolioProject..CovidDeaths
Where continent is not null
GROUP BY date
order by 1


-- Overall Summary in the world
Select SUM(new_cases) as Total_Cases, SUM(cast(new_deaths as int)) as Total_Deaths, SUM(cast(new_deaths as int))/SUM(new_cases)*100 as Percent_Deaths
From PortfolioProject..CovidDeaths
Where continent is not null
--GROUP BY date
--order by date

--Looking at the Population vs Vaccinations available
-- Vaccination data is avaiable from the Covid Vaccinations table but lacks data on population
-- The Covid Deaths table has info on the population so we will join these both tables to get our results

-----------------------------------------------------------
-- Note: Using Partition By with Order by
-- Partition By location sums up all the new vaccinations for all the different countries seperately
-- The new created column displays the total new vaccinations for each row of the individual number of new vaccinations for the day
-- to have a running sum we will order by date along with partition by so that the sum column displays the running sum till that date

--Select DISTINCT deaths.continent, deaths.location, deaths.date, deaths.population,vacc.new_vaccinations
--, SUM(Convert(int, vacc.new_vaccinations)) Over (Partition By deaths.location ORDER BY deaths.location, deaths.date) as Sum_PeopleVaccinated
--From PortfolioProject..CovidDeaths deaths
--Join PortfolioProject..CovidVaccinations vacc
--On deaths.location=vacc.location and deaths.date=vacc.date
--Where deaths.continent is not null
--order by 2,3

------------------------------
-- Finding the percentage of total new vaccinations vs population

-- Now we will be using the new Sum_PeopleVccinated column to find a new metric
-- But we cant simply use this column directly to make a new column for the new metric since it is also a newly created column
-- Hnece we will use a CTE

--Using CTE
-- Note : Order by clause cannot be used with CTE 
With PopsVac (Continent, Location, Date, Population, New_Vaccinations, Sum_PeopleVccinated)
as
(
Select DISTINCT deaths.continent, deaths.location, deaths.date, deaths.population,vacc.new_vaccinations
, SUM(Convert(int, vacc.new_vaccinations)) Over (Partition By deaths.location ORDER BY deaths.location, deaths.date)
From PortfolioProject..CovidDeaths deaths
Join PortfolioProject..CovidVaccinations vacc
On deaths.location=vacc.location and deaths.date=vacc.date
Where deaths.continent is not null
--order by 2,3
)
-- below lines of code will be used to display all the columns along with the one for the new metric we want
Select *, (Sum_PeopleVccinated/Population)*100 as PopvsVac
From PopsVac
-------------------------------------------
-- TEMP TABLE

--Notes on Temp table:---------------
--Creating a new temp table with 
-- 3 columns name, stuent_id and grade
Create Table #Temp
(Name nvarchar(255),
Student_ID numeric,
Grade nvarchar(255)
)

-- Inserting values in a row of temp table
Insert into #Temp
Values ('Shaheer', 05573, 'A')

-- displaying the inserted values
Select *
From #Temp

---------------------------

-- Drop table added so that we dont have to delete the entire table everytime an alteration is made to it
-- b/c we cant just alter a temp table normally
Drop table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,

```sql
New_vaccinations numeric,
Sum_People_Vaccinated numeric
)



Insert into #PercentPopulationVaccinated
Select DISTINCT deaths.continent, deaths.location, deaths.date, deaths.population,vacc.new_vaccinations
, SUM(Convert(int, vacc.new_vaccinations)) Over (Partition By deaths.location ORDER BY deaths.location, deaths.date)
From PortfolioProject..CovidDeaths deaths
Join PortfolioProject..CovidVaccinations vacc
On deaths.location=vacc.location and deaths.date=vacc.date
--Where deaths.continent is not null

Select *
From #PercentPopulationVaccinated



-- Creating view to be used later for visualiing data
Create view Percent_Population_Vaccinated as
Select DISTINCT deaths.continent, deaths.location, deaths.date, deaths.population,vacc.new_vaccinations
, SUM(Convert(int, vacc.new_vaccinations)) Over (Partition By deaths.location ORDER BY deaths.location, deaths.date) as SumPeopleVaccinated
From PortfolioProject..CovidDeaths deaths
Join PortfolioProject..CovidVaccinations vacc
On deaths.location=vacc.location and deaths.date=vacc.date
Where deaths.continent is not null

Select *
From Percent_Population_Vaccinated
