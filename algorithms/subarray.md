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
    * 分治策略
        * 最大子序列可能出现在三处：
            * 左半部分
            * 右半部分
            * 左右两半部分
    * 时间复杂度T(N)=2T(N/2)+O(N)
        * T(N)=Nlog(N)
    ```java
    public class Solution {
        /*
        * @param nums: A list of integers
        * @return: A integer indicate the sum of max subarray
        */
        public int maxSubArray(int[] nums) {
            return maxSumRec(nums,0,nums.length-1);
        }
        
        public int maxSumRec(int[] nums, int left, int right) {
            
            if(left==right) 
                return nums[left];
            
            
            int center = (left+right)/2;
            
            // 注意此处是left，不是0
            int maxLeftSum = maxSumRec(nums,left,center);
            // 注意此处是right，不是nums.length-1
            int maxRightSum = maxSumRec(nums,center+1,right);
            
            int maxLeftBorderSum = nums[center];
            int leftBorderSum = 0;
            for(int i=center;i>=left;i--) {
                leftBorderSum +=nums[i];
                if(leftBorderSum > maxLeftBorderSum)
                    maxLeftBorderSum = leftBorderSum;
            }
            
            
            int maxRightBorderSum = nums[center+1];
            int rightBorderSum = 0;
            for(int i=center+1;i<=right;i++) {
                rightBorderSum +=nums[i];
                if(rightBorderSum > maxRightBorderSum) 
                    maxRightBorderSum = rightBorderSum;
            }
            
            return max(maxLeftSum,maxRightSum,maxLeftBorderSum+maxRightBorderSum);
            
        }
        
        public int max(int left,int right,int center) {
            int max = left;
            if (left < right)
                max = right;
            if (max < center)
                max = center;
            return max;
            
        }
    }
    ```
* 解法4：
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
