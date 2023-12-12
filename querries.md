Database Querries
================
Mishan Phiri
2023-12-07
# Brief
The provided database contained information on the movie industry, with all actors and the necessary information that needs to be captured in a movie context. Querries used to extract information from the database:
![Database](images/datatbase.png)

1. **For each gender, show how many actors there are, who were born between 1940 and 1970**  
   SELECT ActorGenderId,Count(ActorId) as [Gender Count]
   FROM tblActor
   WHERE Year(ActorDob) between 1940 and 1970
   Group by ActorGenderId;
2. **Show all film studios whose total budget is more than 70000000 and those films were released after September**  
    SELECT FilmStudioID,Sum(FilmBudgetDollars) as [Total Budget]
    FROM tblFilm
    WHERE month(FilmReleaseDate) >9
    Group by FilmStudioID
    Having Sum(FilmBudgetDollars)>70000000;
3. **For each employee show the total sales they processed and the average taxes. Only show the details of employees whose average taxes is 3 or more.**  
    SELECT DirectorName, DirectorDob,Count(FilmId) as [Number of Films]
    FROM tblFilm, tblDirector
    WHERE tblDirector.DirectorId=tblFilm.FilmDirectorID
    Group by DirectorName,DirectorDob
    Having Count (FilmId)>4
    Order by  Count(FilmId) DESC;
  4. **Use the insert query to insert data records from the tblCountrySADEC Table into the tblCountry Table**  
    INSERT INTO tblCountry
    SELECT *
    FROM tblCountrySADEC
    WHERE CountryId IN (SELECT CountryId FROM tblCountrySADEC WHERE CountryName not like 'S*')
5. **Write a query to increase the budget of all films released between 1933 and 1970 and 
whose FilmRunTimeMinutes is more than the average FilmRunTimeMinutes by 50%**  
    UPDATE tblFilm
    SET FilmBudgetDollars = FilmBudgetDollars + (FilmBudgetDollars *0.5)
    WHERE Year (FilmReleaseDate) BETWEEN 1933 AND 1970 AND 
    FilmRunTimeMinutes >(SELECT AVG(FilmRunTimeMinutes) FROM tblFilm);
6. **Write a query to delete the record in tblFilm that has the most film budget.**  
    DELETE *
    FROM tblFilm
    WHERE FilmBudgetDollars = (SELECT MAX(FilmBudgetDollars) FROM tblFilm);
