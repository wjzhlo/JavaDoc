# LeetCode 刷题总结

## 一、滑动窗口类型

```
class MedianFinder {

    private List<Integer> store;

    /** initialize your data structure here. */
    public MedianFinder() {
        this.store = new ArrayList<>();
    }

    // 添加数据流
    // 使用二分法，维护有一个有序的数组
    public void addNum(int num) {
        int end = Integer.max(0,this.store.size()-1);
        int start = 0;
        int middle = 0;
        while(true) {
            if(this.store.size() == 0) {
                this.store.add(num);
                break;
            }

            middle = (start+end)/2;

            if((end-start)<=1) {
                if(this.store.get(end) < num) {
                    this.store.add(num);
                }  else if(this.store.get(middle) < num) {
                    this.store.add(middle+1, num);
                } else if(this.store.get(middle) > num) {
                    this.store.add(middle, num);
                }
                break;
            }

            // 若小于找到的中间值，则向右继续查找，直到找到最靠右的小于传入值的数
            if(this.store.get(middle) < num) {
                start = Integer.min(end, middle);
            } else if(this.store.get(middle) > num){
                end = Integer.max(0, middle);
            } else {
                // 相等
                this.store.add(start+1, num);
                break;
            }
        }
        return ;
    }

    // 返回中位数
    public double findMedian() {
        int len = this.store.size();

        if (len == 0) {
            return 0.00;
        }

        if(len%2 == 0) {
            double left = Double.valueOf(this.store.get(len/2));
            double right = Double.valueOf(this.store.get((len/2)-1));
            return (left+right)/2;
        }

        return Double.valueOf(this.store.get(len/2));
    }

    public String getStore() {
        return this.store.toString();
    }
}

```

