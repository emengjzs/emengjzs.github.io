





## 365 Design Twitter

- 使用`typedef` / `using` 为容器迭代器定义别名

  ```c++
  typedef vector<Tweet>::reverse_iterator tweets_itr;
  ```

- 优先使用`emplace_back` 插入自定义对象，参数列表同自定义对象的构造函数签名中的参数列表。

- STL提供逆序迭代器`rbegin()`， `rend()`

  ```c++
  news.emplace_back(userData._tweets.rbegin(), userData._tweets.rend());
  ```

- 对一系列可迭代容器进行归并排序时，使用堆进行排序，有必要同时保存每个容器实例的当前迭代指针和结束迭代器`end()`，使用`pair`包装。

- `vector::reserve()` 能够保持容器容量。

- 若需要持有引用，1. 调用的函数返回引用， 2. 赋值给引用

  ```c++
  UserData& getUserDataById(int userId);
  UserData& userData = getUserDataById(userId);
  ```

- 使用`vector`创建堆：

  - `make_heap(vec.begin(), vec.end(), cmp)` 构建堆
  - `pop_heap(vec.begin(), vec.end(), cmp);` 除去最后一个数，对剩余的元素构建堆， 一般之后调用`pop_back`
  - `push_heap(vec.begin(), vec.end(), cmp);` 假设新增一个元素在结尾，对现在堆进行调整，一般之前调用`push_back`

```c++
class Twitter {
    
    struct Tweet {
        Tweet(int time, int id): _time(time), _id(id) { }
        const int _time;
        const int _id;
    };
    
    struct UserData {
        vector<Tweet> _tweets;
        unordered_set<int> _followings;
        typedef vector<Tweet>::reverse_iterator tweets_itr;
        
        inline void postTweet(int time, int tweetId) {
            _tweets.emplace_back(time, tweetId);
        }
        
        inline void follow(int followeeId) {
            _followings.insert(followeeId);
        }
        
        inline void unfollow(int followeeId) {
            _followings.erase(followeeId);
        }
    };
    
    unordered_map<int, UserData> _userDB;
    int _time;
    
public:
    /** Initialize your data structure here. */
    Twitter(): _time(0) { }
    
    /** Compose a new tweet. */
    void postTweet(int userId, int tweetId) {
        getUserDataById(userId).postTweet(_time ++, tweetId);
    }
    
    /** Retrieve the 10 most recent tweet ids in the user's news feed. Each item in the news feed must be posted by users who the user followed or by the user herself. Tweets must be ordered from most recent to least recent. */
    vector<int> getNewsFeed(int userId) {
        vector<pair<UserData::tweets_itr, UserData::tweets_itr>> news;
        
        UserData& userData = getUserDataById(userId);
        for (int following : userData._followings) {
            vector<Tweet>& tweets = getUserDataById(following)._tweets;
            if (! tweets.empty())
                news.emplace_back(tweets.rbegin(), tweets.rend());
        }
        
        if (! userData._tweets.empty()) {
            news.emplace_back(userData._tweets.rbegin(), userData._tweets.rend());
        }
        
        auto cmp =  [](
            const pair<UserData::tweets_itr, UserData::tweets_itr>& o1, 
            const pair<UserData::tweets_itr, UserData::tweets_itr>& o2) {
               return o1.first->_time < o2.first->_time; 
        };
        
        std::make_heap(news.begin(), news.end(), cmp);
        
        const int n = 10;
        vector<int> results;
        results.reserve(n);
        
        for (int i = 0; i < n && ! news.empty(); ++ i) {
            pop_heap(news.begin(), news.end(), cmp);
            auto& tweet_itr = news.back();
            results.push_back(tweet_itr.first->_id);
            tweet_itr.first ++;
            if (tweet_itr.first == tweet_itr.second) {
                news.pop_back();
            }
            else {
                push_heap(news.begin(), news.end(), cmp);
            }
        }
        
        return results;
    }
    
    /** Follower follows a followee. If the operation is invalid, it should be a no-op. */
    void follow(int followerId, int followeeId) {
        if (followerId != followeeId)
            getUserDataById(followerId).follow(followeeId);
    }
    
    /** Follower unfollows a followee. If the operation is invalid, it should be a no-op. */
    void unfollow(int followerId, int followeeId) {
        getUserDataById(followerId).unfollow(followeeId);
    }
    
    
private: 
    inline UserData& getUserDataById(int userId) {
        return _userDB[userId];
    }
};

/**
 * Your Twitter object will be instantiated and called as such:
 * Twitter obj = new Twitter();
 * obj.postTweet(userId,tweetId);
 * vector<int> param_2 = obj.getNewsFeed(userId);
 * obj.follow(followerId,followeeId);
 * obj.unfollow(followerId,followeeId);
 */
```



## 347 Top K Frequent Elements          

- 使用priority_queue构建优先队列（堆）， `push`添加元素、`top`获取最值、`pop`删除最值
- 比较默认为`less<T>()`, 即默认按a1 < a2 < a3 < ...，升序
- 对pair排序默认按第一个元素的升值排序

```c++
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> frequent_table;
        for(int n: nums) {
            frequent_table[n] ++;
        }
        priority_queue<pair<int, int>> queue;
        vector<int> result;
        for (auto& num_count : frequent_table) {
            queue.push(make_pair(num_count.second, num_count.first));
            if (frequent_table.size() - queue.size() < k) {
                result.push_back(queue.top().second);
                queue.pop();
            }
        }
        return result;
    }
};
```