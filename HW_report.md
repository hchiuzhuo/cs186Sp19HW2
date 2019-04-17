#  HW2 report

This project is to implement B+ tree. Although the whole framework are provided and the algorithm is clearly specified, 
I still struggled with several parts. Perhaps that's to remind me that I need to practice my coding skill more often.

* Leafnode

    * get(), getLeftmostLeaf(): 
        
        No specific problems.
        
    * put(): 
        
        When break, the right sibling should update w.t. current right sibling
        
        ```java
        int addIdx;

        for(int i = 0; i < this.keys.size(); i++){
            if(key.compareTo(this.keys.get(i)) < 0){
               addIdx = i;
               break;
            }
        }
        //can ber replaced with helper function  
        addIdx = InnerNode.numLessThan(key, this.keys);  
        
        ```
        
    * bulkLoad():
        
        ```java
        Initially, i use a sorted tree map to keep the input data sorted and contains no duplicated key.
        However, put this in leaf node will force the input data iterator run to the last index at beginning.
        Therefore, i remove this part and keep input data (sorted w.t. duplicated key unhandled)
        //        TreeMap sortedData = new TreeMap();
        //        Pair<DataBox, RecordId> entry;
        //        while (data.hasNext()){
        //            entry = data.next();
        //            if(sortedData.containsKey(entry.getFirst())) throw new BPlusTreeException("Duplicated key");
        //            sortedData.put(entry.getFirst(), entry.getSecond());
        //        }
        //        Iterator sortedDataItr = sortedData.entrySet().iterator();

       ```

* Inner node

    * get(), getLeftmostLeaf():
    
      Initially, I directly return leaf node. That's incorrect because I did not consider recursive case in which the 
      inner node has inner node children.
      
    * put():
      
      Initially, I stuck at how to determine whether a bplusnode is inner node or leaf node.
      I should check helper functions in detail in the beginning, in which `getChild(transaction, idx)`
      has already implemented for me.

      ```java
      BPlusNode next = getChild(transaction, idx);
      ```
      
      Since i ignored using `getChild()` in the beginning, I fall into a recursive hell!!
      I always cannot return to the correct stack layer.
      ```java
      
        this try catch recursive iteration won't return result to current node.
        because: current node -> nxt inner node -> assertion error -> leafnode -> return res to current node. the nxt innernode won't exit on the recuresive path.
        try {
            InnerNode nxt = InnerNode.fromBytes(transaction, metadata, children.get(i).intValue());
            res = nxt.put(transaction, key, rid);
        }catch (AssertionError e) {
            res = LeafNode.fromBytes(transaction, metadata, children.get(i).intValue()).put(transaction,key,rid);
            return res;
        }
 
    * bulkLoad()
    
    The idea of buckload is to get the left most leaf and start to insert data into that leaf, when 
    the leaf split, copy up the split key to inner node. The inner node does not need to get the 
    key of the item in input data list. Instead, always get the rightest leaf to insert.


* BPlusTree

    * get():
    
    No specific problems.
    
    * scanAll(), scanGreaterEqual():
    
    Both of them requires to implement BPlusTreeIterator. The trick is once to get the left most leaf, you should use 
    getRightSibling() to sequentially read the rest leafs without going into inner node. 
    Another helper function that already implemented is leaf will return an iterator of its records.
      
    * put(), bulkload():
    
    put and bulkload are similar. Two things need to be noticed are 1) the innitial root is leaf node, so the first split with 
    copying up to inner node need to be handled in these two functions, and 2) the header page needs to be updated with root's 
    newest page id when the root is split.
    
    
* Others.

   In LeafNode, InnerNode and BPlusTree, always need to sync w.t. disk to keep the data persistent.
   For LeafNode and InnerNode, use sync to sync the page.
   For BPlusTree, use `writeHeader(transaction,this.headerPage)` to sync-up w.t. header page. 