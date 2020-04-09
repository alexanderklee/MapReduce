# Simple MapReduce script to find top movies ratings using the 1990's ratings data set

The goal of this script is to find the top most rated movies in 1998. The data set is provided by IMDB and provides a simple table of
the user id's, ratings and the movie index. Supplemental data also includes a movies table which provides the necessary mapping of movie
titles to movie indices. 

## Simple design

This script uses the MRJob and MRStep Python library to help execute the job on a Hadoop/HDFS node. The process does require a nested
reducer operation as noted in the code sample below:


class RatingsBreakdown(MRJob):
    def steps(self):
        return [
            MRStep(mapper=self.mapper_get_ratings,
                   reducer=self.reducer_count_ratings),
            MRStep(reducer=self.reducer_sorted_output)
        ]

In this case, we simply create a new python class and define our MRStep function in a nested fashion. Also, we define our mapper and
reducer functions here as well, serving as a constructor (I'm a C++ person) of sorts to drive the application. 

Next we will define our mapper and reducer functions. Below we simply extract the data in the user ratings data set into 4 separate
python variables and accumulate a list of movie ID's followed by a simplified counter of 1. This results in a long list of movie ID's
with a value of "1" where this list will have duplicate entires. This is by design because of how the MapReduce logic works. 

    def mapper_get_ratings(self, _, line):
        (userID, movieID, rating, timestamp) = line.split('\t')
        yield movieID, 1
        
Next we define our 1st reducer function where we accumulate and pad values produced by the mapper routine. While there are better 
approaches to doing this (eg., use JSON or another data exchange format), this is a QAD project and so this level of implementation 
was not necessary.

    def reducer_count_ratings(self, key, values):
        yield str(sum(values)).zfill(5), key

Next we will define our 2nd reducer where the results from the 1st reducer job will be passed into this 2nd one. This second reducer
simply iterates over the reduced movie id data set and accumulates/counts all the necessary rating counts. 

    def reducer_sorted_output(self, count, movies):
        for movie in movies:
            yield movie, count

Lastly, we'll need to tell the python interpreter that this module will run as the "main". Again, this is a design decision based on 
the simplicity of this script. By no means is tnis a great program as a whole. 

if __name__ == '__main__':
    RatingsBreakdown.run()

