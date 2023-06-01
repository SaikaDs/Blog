最近工作写了不少qt，然后意识到Unity的属性面板不也是可定制的吗，所以打算学一下。   
### CustomEditor
bing了一下了解到Unity中要自定义属性面板，主要是靠CustomEditor(自定义编辑器)这个类。每个CustomEditor类可以包住一个Monobehaviour，然后在OnInspectorGUI方法里定制如何绘制属性面板。  
参考博客https://blog.csdn.net/woodengm/article/details/120952669  
### 实际使用
最近在写怪物的接触攻击，即角色碰到怪物就受击。实现方式是在怪物下挂一个子物体，加一个BoxCollider，设为trigger，然后在碰撞事件里调用攻击逻辑。  
前提是因为各种原因怪物碰撞盒跟这个攻击区域碰撞和需要是两个东西。而一般来说，碰撞攻击区域就应该跟怪物本身碰撞盒一样，所以需要手动填成一样的区域。  
就想能不能加一个一键复制按钮，把怪物本体的碰撞盒参数复制到攻击区域碰撞盒上。  
代码如下：
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

[RequireComponent(typeof(BoxCollider2D))]
public class TouchAttack : MonoBehaviour
{
    MonsterAttr attr;

    private void Awake()
    {
        attr = GetComponentInParent<MonsterAttr>();
    }

    private void OnTriggerEnter2D(Collider2D collision)
    {
        AttackInfo info = new AttackInfo((attr.TouchATK), transform.position, Vector2.zero, 0, 0, AtkMaterial.Knock, 0);
        collision.gameObject.GetComponent<Life>().Beat(info);
    }

    public void CopyParentCollider()
    {
        var self_box = GetComponent<BoxCollider2D>();
        var parent_box = transform.parent.GetComponent<BoxCollider2D>();
        self_box.size = parent_box.size;
        self_box.offset = parent_box.offset;
    }
}

[CustomEditor(typeof(TouchAttack))]
public class TouchAttackEditor : Editor
{
    public override void OnInspectorGUI()
    {
        base.OnInspectorGUI();

        TouchAttack _target = target as TouchAttack;
        if (GUILayout.Button("复制BoxCollider"))
        {
            _target.CopyParentCollider();
            Debug.Log("复制父对象BoxCollider完成");
        }
    }
}
```
- 其中，target是内置的对象，应该就是表示装饰的目标对象，直接获取来是Object类型，因此需要做个转换。  
- if (GUILayout.Button(""))这种写法有点像语法糖，应该是把Button设置和Button回调一起包含了，太酷了。之后详细了解下  
- 实际测试下来，CustomEditor是可以与目标Monobehaviour处于同一个脚本文件里，省事了。
