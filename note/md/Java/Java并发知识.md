# Synchronized

* 锁定的对象的属性可以变化，但是锁的引用不能，否则保存在对象中的锁定信息将无法保留，导致同步失败
* 不要以字符串作为锁定对象
* 为了避免脏读，对读方法也要加锁
* synchronized锁是可重入的(必须是同一个线程)

# Volatile

* 保证数据的可见性，**但是不能保证数据的同步**，原因在于当工作内存中的数据进行了read或load就不会再改变了