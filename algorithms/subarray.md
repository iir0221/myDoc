# 最大子序列和问题
* 问题描述：对于输入-2,11,-4,13,-5,-2...求其子序列最大的和

* 解法1:
    ```java
    public class Solution {
        /*
        * @param nums: A list of integers
        * @return: A integer indicate the sum of max subarray
        * @Time: O(N)的立方
        */
        public int maxSubArray(int[] nums) {
            int maxSum = nums[0];
            for(int i=0;i<nums.length;i++) {
                
                for(int j=i;j<nums.length;j++) {
                    // 计算每次i~j的和
                    int sum = 0;
                    for(int k=i;k<=j;k++) {
                        sum +=nums[k];
                    }
                    
                    if(sum>maxSum)
                        maxSum = sum;
                    
                }
                
            }
            
            return maxSum;
        }
    }
    ```
* 解法2：
    ```java
    public class Solution {
        /*
        * @param nums: A list of integers
        * @return: A integer indicate the sum of max subarray
        * @Time: O(N)的平方
        */
        public int maxSubArray(int[] nums) {
            int maxSum = nums[0];
            for(int i=0;i<nums.length;i++) {
                
                int sum = 0;
                // 选出由j~i的最大和
                for(int j=i;j<nums.length;j++) {
                    
                    sum +=nums[j];
                    if(sum>maxSum)
                        maxSum = sum;
                }
            }
            return maxSum;
        }
    }
    ```
* 解法3：
    * 最优解
    * 我们不需要知道最佳子序列的位置
    * 如果a[i]是负的，那么它不可能代表最优序列的起点
    * 同理，任何负的子序列也不可能是最优子序列的前缀
    ```java
    public class Solution {
        /*
        * @param nums: A list of integers
        * @return: A integer indicate the sum of max subarray
        * @Time: O(N)
        */
        public int maxSubArray(int[] nums) {
            int maxSum = nums[0];int sum=0;
            for(int i=0;i<nums.length;i++) {
                
                sum+=nums[i];
                if(sum>maxSum)
                    maxSum = sum;
                else if(sum<0)
                    sum=0;
            }
            
            return maxSum;
        }
    }
    ```
* 解法4：
    * 分治策略
    * 利用递归
    * NlogN