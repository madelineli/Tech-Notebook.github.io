---
layout: post
title: "Muli-Column Sorting Using Apache Commons ComparatorChain"
published: true
---

Brief Intro
-----------

Apache Commons project offers a rich set of utilities to make developing in Java less tedious. (IMO, it's one of the most useful tool sets out there.) The ComparatorChain class is one such tool. It wraps one or more Comparators in sequence and calls each Comparator in sequence. When sorting is performed, the ComparatorChain calls each Comparator in the order it was added until a Comparator returns a non-zero result or the ComparatorChain is exhausted. It's very much like multi-column sorting in SQL.

First, one needs to know the difference between a Comparator vs. a Comparable in Java. Here is a short description.

A *Comparable* object is used to compare an object with another instance of the same class. An easier way to remember is to think of how one uses String's equals() method for comparing a string to another string.

A *Comparator* object is capable of comparing two objects which can be instances of the same class or instances of separate classes.

As the name suggests, the CompratorChain works with classes that implement the Comparator interface. It relies on the compare() method of the Comparator classes. To get the comparsion going, you need to 

1. Create a Comparator class. Instances of the Comparator are added to ComparatorChain.
2. Use java.util.Collections's sort method to perform the sort.


Example
-------

 I have a result collection of JSON objects. I want to sort the JSON objects by the date field in the descending order, followed by the title field in the ascending order.  The following shows an example search result:

 
    "results":[
            { "Id": "1234", "Date": "2014-08-16", "Relevance": "0.2345", "Title": "Trouble in South China Sea"},
            { "Id": "3450", "Date": "2008-09-15", "Relevance": "0.7888", "Title": "Gaza Under Fire"},
            { "Id": "4560", "Date": "2014-08-16", "Relevance": "0.3333", "Title": "Fergusion Riot"}
     ]


(1) Create a class to specify Sort crieteria:

    public class SearchSort {

       public Static enum Direction {
         asc,
         desc;

     
       public static Direction fromString(String inDirection) {
         for (Direction direction: values()) {
           if (StringUtils.equalsIgnoreCase(inDiection, direction.name())) {
              return direction;
           }
         }
         return null;
       }

       private String field;
       private Direction direction;

       public SearchSort() {}
       public SearchSort(String field, Direction direction) {
         this.field = field;
         this.direction = direction;
       }
       public setField(String field) { this.field = field;}
       public String getField() { return this.field; }
       public setDiriection(Direction dir) { this.direction = dir;}
       public getDirection() { return this.direction;}
   
    }     

(2) Create a class that implements the Comparator interface:

    public class searchResultComparator implements Comparator<JsonNode> {

      private SearchSort searchSort;

      public SearchResultComparator(SearchSort searchSort) throws InvalidSortException {
           if (searchSort == null || searchSort.getField() == null || searchSort.getField))
             throw new InvalidSortException("SearchSort used to specify the sort criteria cannot be null!");
           this searchSort = searchSort;

      }

      public int compare(JsonNode node1, JsonNode node2) {
         if (node1 == null && node2 == null) return 0;
         if (node1 == null) return -1;
         if (node2 == null) return 1;

         if (isNull(node1, searchSort.getField())) {
           if (isNull(node1, searchSort.getField())) return 0;
           return -1;

         }
         else {
           if (isNull(node2, searchSearch.getField())) return 1;

         }
         
         if (node1.get(searchSort.getField()).isDouble()) {
           Double val1, val2;
           val1 = node1.get(searchSort.getField()).asDouble();
           val2 = node2.get(searchSort.getField()).asDouble();
           return va11.compareTo(val2);
         }

         ....

         if (node1.get(searchSort.getField()).isInt()) {
           Integer val1, val2;
           val1 = node1.get(searchSort.getField()).asInt();
           val2 = node2.get(searchSort.getField()).asInt();
           return val1.compareTo(val2);

         }

         // Attempt to sort w/date and then text if it does not work
         Date date1, date2;
         String val1, val2;

         val1 = node1.get(searchSort.getField()).asText().toLowerCase();         
         val2 = node2.get(searchSort.getField()).asText().toLowerCase();
         
         // try to parse the text to see if it's a date field. If not, fail gracefully (swallow the exception).
         // If it's a date filed, then convert val to date object before comparision

         int cmp;
         DateFormat DATE_TIME_FORMAT = new SimpleDateFormat("yyyy-MM-dd:HH:mm:ss");
         DateFormat DATE_NO_TIME_FORMAT = new SimpleDateFormat("yyyy-MM-dd");

         try {
            date1 = DATE_TIME_FORMAT.parse(val1);
            date2 = DATE_TIME_FORMAT.parse(val2);
            cmp = date1.compareTo(date2);
         }
         catch(ParseException pe) {
            try {
               date1 = DATE_NO_TIME_FORMAT.parse(val1);
               date2 = DATE_NO_TIME_FORMAT.parse(val2);
               cmp = date1.compareTo(date2);
            }
            catch(ParseException pe2) {

              // gracefully ignore. Compare the two as string
              cmp = val1.comepareTo(val2);
            }
         }
         
         return cmp;

       }

       private boolean isNull(JsonNode node, String field) {
           if (node.get(field) == null || node.get(field).isNull() || node.get(field).asText().equals("null")) return true;
           return fals;
       }

    }

(3) Create a collection of searchSort objects for search Criteria:

    ArrayList<SearchSort> searchSortList = new ArrayList<SearchSort>)();
    searchSortList.add(new SearchSort("Date", Direction.asc);
    SearchSortList.add(new SearchSort("Title", Direction.asc);

(3) Create a ComparatorChain object to do the comparisons:

    org.apache.commons.collections.compartor.ComparatorChain  chain = new org.apache.commons.collections.comparator.ComparatorChain();

    for (SearchSort searchSort : searchSortList) {
      try {
         SearchResultComparator comparator = new SearchResultComparator(searchSort);
         chain.addComparator(comparator, searchSort.getDirection() == SearchSort.Diection.desc); // false: ascending order, true: descending order
      catch (InvalidSortException ise) {
         log.error("The SearchSort object is null and cannot be used for sorting.");
      }
    }

    // Now sort the collection! supposed that our results are in a List<JsonNode> results
    Collections.sort(results, chain);  //Collections is part of java.util package.


